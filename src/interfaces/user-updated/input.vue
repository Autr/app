<template>
  <div v-if="userInfo">
    <v-avatar
      v-if="options.display !== 'name'"
      v-tooltip="options.display === 'avatar' ? displayValue : null"
      class="display-user"
      :size="28"
      :src="src"
      :alt="displayValue"
      color="blue-grey-300"
    ></v-avatar>
    <span v-if="options.display !== 'avatar'" class="label gray type-label">
      <div>{{ displayValue }}</div>
    </span>
  </div>
  <div v-else-if="newItem" class="gray type-label">
    {{ $t("interfaces-user-updated-you") }}
  </div>
  <div v-else class="gray type-label">
    {{ $t("interfaces-user-updated-unknown") }}
  </div>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";

export default {
  mixins: [mixin],
  computed: {
    userInfo() {
      if (!this.value) return null;
      if (typeof this.value === "object") {
        if (this.value.first_name) {
          return this.value;
        } else {
          return this.$store.state.users[this.value.id];
        }
      }
      return this.$store.state.users[this.value];
    },
    displayValue() {
      return this.$helpers.micromustache.render(this.options.template, this.userInfo);
    },
    src() {
      if (!this.userInfo.avatar) return null;
      return this.userInfo.avatar.data.thumbnails[0].url;
    }
  }
};
</script>

<style lang="scss" scoped>
.display-user {
  width: max-content;
  display: inline-block;
  vertical-align: top;
}
.label {
  display: inline-block;
  margin-left: 8px;
  height: 28px;
  div {
    margin-top: 6px;
  }
}
.gray {
  color: var(--blue-grey-300);
}
</style>
