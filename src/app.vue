<template>
  <div v-if="localeLoaded && hydratingError" class="error">
    <v-error
      icon="warning"
      :title="$t('server_error')"
      :body="$t('server_error_copy')"
      color="danger"
    />
    <p class="try-again">
      Try again later or
      <router-link to="/logout">login to a different project</router-link>
    </p>
  </div>

  <div v-else-if="localeLoaded && extensionError" class="error">
    <v-error
      icon="extension"
      :title="$t('extensions_missing')"
      :body="$t('extensions_missing_copy')"
      color="warning"
    />
  </div>

  <div
    v-else-if="localeLoaded && !publicRoute"
    :style="{
      '--brand': `var(--${color})`
    }"
  >
    <div v-if="hydrated" class="directus">
      <v-nav-sidebar />
      <router-view class="page-root" />
    </div>

    <loader v-else area="full-page" />

    <portal-target name="modal" multiple />

    <v-notification />
  </div>

  <div v-else-if="localeLoaded" class="public">
    <router-view />
    <v-notification />
  </div>
</template>

<script>
import { mapState } from "vuex";
import VError from "./components/error.vue";
import { TOGGLE_NAV } from "./store/mutation-types";
import VNavSidebar from "./components/sidebars/nav-sidebar/nav-sidebar.vue";
import VNotification from "./components/notifications/notifications.vue";
import { loadLanguageAsync, availableLanguages } from "./lang";

export default {
  name: "Directus",
  metaInfo: {
    title: "Directus"
  },
  components: {
    VError,
    VNavSidebar,
    VNotification
  },
  data() {
    return {
      localeLoaded: false
    };
  },
  computed: {
    ...mapState({
      color: state =>
        state.settings.values.project_color ||
        getComputedStyle(document.documentElement)
          .getPropertyValue("--brand")
          .trim(),
      infoActive: state => state.sidebars.info
    }),
    publicRoute() {
      return this.$route.meta.publicRoute || false;
    },
    hydrated() {
      return this.$store.state.hydrated || false;
    },
    hydratingError() {
      return this.$store.state.hydratingError;
    },
    extensionError() {
      if (!this.hydrated) return null;

      const extensions = this.$store.state.extensions;

      if (
        Object.values(extensions.interfaces).length === 0 &&
        Object.values(extensions.layouts).length === 0 &&
        Object.values(extensions.pages).length === 0
      ) {
        return true;
      }

      return false;
    }
  },
  watch: {
    $route() {
      this.bodyClass();
      this.$store.commit(TOGGLE_NAV, false);
    },
    infoActive(visible) {
      this.toggleInfoSidebarBodyClass(visible);
    },
    hydratingError(newVal) {
      if (newVal) {
        document.body.classList.add("no-padding");
      }
    }
  },
  created() {
    this.bodyClass();

    const shouldLoadLocale =
      window.__DirectusConfig__ &&
      window.__DirectusConfig__.defaultLocale &&
      window.__DirectusConfig__.defaultLocale !== "en-US";

    if (shouldLoadLocale) {
      let { defaultLocale } = window.__DirectusConfig__;

      if (defaultLocale === "auto") {
        let locale;

        if (navigator.languages != undefined) {
          locale = navigator.languages[0];
        } else {
          locale = navigator.language;
        }

        if (locale.length !== 5) {
          // try finding a locale that fits the current language
          const locales = Object.keys(availableLanguages);
          locale = locales.find(l => l.startsWith(locale)) || "en-US";
        }

        defaultLocale = locale;
      }

      loadLanguageAsync(defaultLocale).then(() => {
        this.localeLoaded = true;
      });
    } else {
      this.localeLoaded = true;
    }
  },
  methods: {
    bodyClass() {
      if (this.publicRoute) {
        document.body.classList.add("no-padding");
      } else {
        document.body.classList.remove("no-padding");
      }

      this.toggleInfoSidebarBodyClass();
    },
    keepEditing() {
      this.$router.push(
        `/collections/${this.$store.state.edits.collection}/${this.$store.state.edits.primaryKey}`
      );
    },
    discardChanges() {
      this.$store.dispatch("discardChanges");
    },

    toggleInfoSidebarBodyClass(visible = null) {
      if (visible === null) visible = this.infoActive;

      const className =
        this.$route.meta && this.$route.meta.infoSidebarWidth === "wide"
          ? "info-wide-active"
          : "info-active";

      if (visible) {
        document.body.classList.add(className);
      } else {
        document.body.classList.remove("info-wide-active");
        document.body.classList.remove("info-active");
      }
    }
  }
};
</script>

<style lang="scss">
body.no-padding {
  padding: 0 !important;

  &::before {
    display: none;
  }
}

body.info-active {
  padding-right: 284px;
}

body.info-wide-active {
  padding-right: 284px;
}
</style>

<style lang="scss" scoped>
.page-root {
  background-color: var(--page-background-color);
}
.error {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.try-again {
  position: absolute;
  bottom: 40px;
}

.public {
  height: 100%;
}
</style>
