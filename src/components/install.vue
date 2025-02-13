<template>
  <v-modal
    :tabs="tabs"
    :buttons="buttons"
    :active-tab="activeTab"
    action-required
    @tab="activeTab = $event"
    @next="nextTab"
  >
    <template slot="project">
      <h1 class="style-0">{{ $t("project_info") }}</h1>
      <p class="subtext">{{ $t("project_info_copy") }}</p>

      <form @submit.prevent="nextTab">
        <label>
          {{ $t("project_name") }}
          <v-input
            id="project-name"
            ref="projectName"
            v-model="values.project_name"
            class="input"
            autofocus
          />
        </label>
        <label>
          {{ $t("project_key") }}
          <v-input id="environment" class="input" disabled value="Default ( _ )" />
        </label>
        <label>
          {{ $t("admin_email") }}
          <v-input id="admin-email" v-model="values.user_email" class="input" type="email" />
        </label>
        <label>
          {{ $t("admin_password") }}
          <v-input
            id="admin-password"
            v-model="values.user_password"
            class="input"
            type="password"
            autocomplete="new-password"
          />
        </label>
        <input type="submit" class="hidden" />
      </form>
    </template>

    <template slot="database">
      <h1 class="style-0">{{ $t("database_connection") }}</h1>
      <p class="subtext">{{ $t("database_connection_copy") }}</p>
      <form @submit.prevent="nextTab">
        <label>
          {{ $t("host") }}
          <v-input id="db_host" v-model="values.db_host" class="input" />
        </label>
        <label>
          {{ $t("port") }}
          <v-input id="db_port" v-model="values.db_port" class="input" />
        </label>
        <label>
          {{ $t("db_user") }}
          <v-input id="db_user" v-model="values.db_user" class="input" />
        </label>
        <label>
          {{ $t("db_password") }}
          <v-input id="db_password" v-model="values.db_password" type="password" class="input" />
        </label>
        <label>
          {{ $t("db_name") }}
          <v-input id="db_name" v-model="values.db_name" class="input" />
        </label>
        <label>
          {{ $t("db_type") }}
          <v-input id="db_type" class="input" disabled value="MySQL & Variants" />
        </label>
        <input type="submit" class="hidden" />
      </form>
    </template>
  </v-modal>
</template>

<script>
export default {
  name: "VInstall",
  props: {
    saving: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      activeTab: "project",

      values: {
        db_host: "localhost",
        db_port: 3306,
        db_name: "directus",
        db_user: "",
        db_password: null,
        user_email: null,
        user_password: null,
        project_name: "Directus",
        cors_enabled: true
      }
    };
  },
  computed: {
    databaseDisabled() {
      const { isEmpty } = _;
      const { project_name, user_email, user_password } = this.values;

      return isEmpty(project_name) || isEmpty(user_email) || isEmpty(user_password);
    },
    schemaDisabled() {
      const { isEmpty } = _;
      const { db_host, db_port, db_name, db_user, db_password } = this.values;

      return (
        isEmpty(db_host) ||
        isEmpty(db_port) ||
        isEmpty(db_name) ||
        isEmpty(db_user) ||
        isEmpty(db_password)
      );
    },
    buttons() {
      let disabled = false;

      if (this.activeTab === "project" && this.databaseDisabled) disabled = true;

      return {
        next: {
          disabled,
          text: this.activeTab === "database" ? this.$t("save") : this.$t("next"),
          loading: this.saving
        }
      };
    },
    tabs() {
      return {
        project: {
          text: this.$t("project")
        },
        database: {
          text: this.$t("database"),
          disabled: this.databaseDisabled
        }
      };
    }
  },
  methods: {
    nextTab() {
      switch (this.activeTab) {
        case "project":
          this.activeTab = "database";
          break;
        case "database":
        default:
          this.save();
          break;
      }
    },
    save() {
      this.$emit("install", this.values);
    }
  }
};
</script>

<style lang="scss" scoped>
.hidden {
  display: none;
}

.tab {
  padding: 30px;

  .style-0 {
    max-width: 80%;
    margin-top: 20px;
    margin-bottom: 20px;
  }

  p.subtext {
    max-width: 400px;
    font-size: 16px;
    color: var(--blue-grey-300);
    line-height: 26px;
    font-weight: 400;
  }
}

.tabs {
  display: flex;
  padding: 0;
  list-style: none;
  justify-content: center;
  border-bottom: 1px solid var(--blue-grey-50);
  position: sticky;
  top: 0;
  background-color: var(--white);
  z-index: +1;

  button {
    flex-grow: 1;
    flex-shrink: 1;
    max-width: 120px;
    flex-basis: 120px;
    height: 50px;
    position: relative;
    color: var(--blue-grey-400);

    text-decoration: none;
    text-transform: uppercase;
    font-size: 12px;
    font-weight: 700;
    position: relative;

    &:hover {
      color: var(--blue-grey-800);
    }

    &::after {
      content: "";
      display: block;
      width: 100%;
      position: absolute;
      height: 3px;
      bottom: -2px;
      background-color: var(--blue-grey-900);
      transform: scaleY(0);
      transition: transform var(--fast) var(--transition-out);
    }

    &.active {
      color: var(--blue-grey-900);

      &::after {
        transform: scaleY(1);
        transition: transform var(--fast) var(--transition-in);
      }
    }

    &[disabled] {
      color: var(--blue-grey-200);
      cursor: not-allowed;
    }
  }
}

form {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 20px;
  padding: 40px 0 40px;

  .input {
    margin-top: 10px;
  }
}
</style>
