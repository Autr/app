<template>
  <div class="settings-roles">
    <v-header :breadcrumb="breadcrumb" icon-link="/settings" settings="true">
      <template slot="buttons">
        <v-header-button
          key="add"
          icon="add"
          background-color="button-primary-background-color"
          icon-color="button-primary-text-color"
          :label="$t('new')"
          @click="addNew = true"
        />
      </template>
    </v-header>
    <v-table :items="items" :columns="fields" primary-key-field="collection" link="__link__" />

    <portal v-if="addNew" to="modal">
      <v-prompt
        v-model="newName"
        :confirm-text="$t('create')"
        :message="$t('create_role')"
        :placeholder="$t('enter_role_name')"
        :loading="adding"
        required
        @cancel="addNew = false"
        @confirm="add"
      />
    </portal>
  </div>
</template>

<script>
import api from "../../api";

export default {
  name: "SettingsRoles",
  metaInfo() {
    return {
      title: `${this.$t("settings")} | ${this.$t("roles")}`
    };
  },
  data() {
    return {
      error: null,
      roles: [],
      adding: false,
      addNew: false,
      newName: ""
    };
  },
  computed: {
    breadcrumb() {
      return [
        {
          name: this.$t("settings"),
          path: "/settings"
        },
        {
          name: this.$t("roles"),
          path: "/settings/roles"
        }
      ];
    },
    items() {
      return this.roles.map(role => ({
        ...role,
        __link__: `/settings/roles/${role.id}`
      }));
    },
    fields() {
      return [
        {
          field: "name",
          name: this.$t("name")
        },
        {
          field: "description",
          name: this.$t("description")
        }
      ];
    }
  },
  beforeRouteEnter(to, from, next) {
    api
      .getRoles()
      .then(res => res.data)
      .then(roles => {
        next(vm => {
          vm.$data.roles = roles;
        });
      })
      .catch(error => {
        this.$events.emit("error", {
          notify: this.$t("something_went_wrong_body"),
          error
        });
        next(vm => {
          vm.$data.error = error;
        });
      });
  },
  methods: {
    add() {
      this.adding = true;

      const id = this.$helpers.shortid.generate();
      this.$store.dispatch("loadingStart", { id });

      this.$api
        .createRole({
          name: this.newName
        })
        .then(res => res.data)
        .then(role => {
          this.$store.dispatch("loadingFinished", id);
          this.$router.push(`/settings/roles/${role.id}`);
        })
        .catch(error => {
          this.adding = false;
          this.$store.dispatch("loadingFinished", id);
          this.$events.emit("error", {
            notify: this.$t("something_went_wrong_body"),
            error
          });
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.settings-roles {
  padding: 0 32px var(--page-padding-bottom);
}
</style>
