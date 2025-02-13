<template>
  <div
    class="project-switcher"
    :class="{
      'is-active': $store.state.auth.projectName !== selectionName || !$store.state.auth.project,
      'has-error': !$store.state.auth.project
    }"
  >
    <div
      v-tooltip.left="{
        content:
          (!!$store.state.auth.url ? $store.state.auth.url : 'No connection') +
          `<br>${$t('latency')}: ${
            !!$store.state.auth.url
              ? $n(Math.round($store.state.latency[$store.state.latency.length - 1].latency))
              : ' - '
          }ms`,
        boundariesElement: 'body'
      }"
      class="content"
      :class="{
        slow: $store.getters.signalStrength === 1,
        disconnected: $store.getters.signalStrength === 0
      }"
    >
      <v-signal class="icon" />
      <span class="no-wrap project-name">
        {{ selectionName ? selectionName : $store.state.auth.projectName }}
      </span>
      <v-icon v-if="Object.keys(urls).length > 1" class="chevron" name="expand_more" />
      <select v-if="Object.keys(urls).length > 1" :value="currentUrl" @change.prevent="changeUrl">
        <option
          v-for="(name, url) in urls"
          :key="name + url"
          :name="name"
          :value="url"
          :selected="url === currentUrl || url + '/' === currentUrl"
          @click="changeUrl"
        >
          {{ name }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import VSignal from "../../signal.vue";

export default {
  name: "ProjectSwitcher",
  components: {
    VSignal
  },
  data() {
    return {
      active: false,
      selectionUrl: null,
      selectionName: ""
    };
  },
  computed: {
    urls() {
      return _.mapKeys(window.__DirectusConfig__.api, (val, key) =>
        key.endsWith("/") === false ? key + "/" : key
      );
    },
    currentUrl() {
      return this.$store.state.auth.url + "/" + this.$store.state.auth.project + "/";
    }
  },
  methods: {
    changeUrl(event) {
      const newUrl = event.target.value;
      const newName = window.__DirectusConfig__.api[newUrl]
        ? window.__DirectusConfig__.api[newUrl]
        : this.$store.state.auth.projectName;

      this.selectionUrl = newUrl;
      this.selectionName = newName;

      this.$store
        .dispatch("switchProject", {
          projectName: newName,
          url: newUrl
        })
        .then(() => this.$store.dispatch("changeAPI", newUrl));
    }
  }
};
</script>

<style lang="scss" scoped>
.project-switcher > div {
  height: var(--header-height);
  width: calc(100% + 40px);
  display: flex;
  align-items: center;
  margin: 0 -20px 20px;
  padding: 0 30px;
  position: relative;
  background-color: var(--sidebar-background-color-alt);

  &:hover {
    .chevron {
      opacity: 1;
    }
  }

  .content {
    padding: 8px 0 8px 10px;
  }

  &.slow {
    svg {
      transition: color 0.25s ease-in-out, fill 0.25s ease-in-out;
    }

    &.slow {
      svg {
        fill: var(--warning);
      }
    }

    &.disconnected {
      svg {
        fill: var(--danger);
      }
    }

    svg {
      fill: var(--sidebar-text-color);
    }

    i {
      color: var(--sidebar-text-color);
    }

    &.has-error {
      > div {
        svg {
          fill: var(--red);
        }
      }
      span {
        color: var(--red);
        + i {
          color: var(--red);
        }
      }
    }
  }

  .icon {
    flex-shrink: 0;
    width: 21px;
    height: 24px;
    margin-right: 21px;
    color: var(--sidebar-text-color);
    fill: var(--sidebar-text-color);
  }

  .project-name {
    padding-right: 12px;
  }

  .chevron {
    position: absolute;
    right: 10px;
    color: var(--sidebar-text-color);
    opacity: 0;
    transition: all var(--fast) var(--transition);
  }

  select {
    position: absolute;
    opacity: 0;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    width: 100%;
    height: 100%;
    cursor: pointer;
  }
}
</style>
