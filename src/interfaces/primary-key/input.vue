<template>
  <div>
    <v-input
      class="input"
      :maxlength="length"
      :type="inputType"
      :readonly="!newItem"
      :value="value"
      @input="$emit('input', $event)"
    ></v-input>
    <span v-if="!newItem" class="immutable">
      {{ $t("interfaces-primary-key-immutable") }}
    </span>
  </div>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";

export default {
  mixins: [mixin],
  computed: {
    inputType() {
      if (this.type === "string") {
        return "text";
      }

      return "number";
    }
  }
};
</script>

<style lang="scss" scoped>
.input {
  width: 100%;
  max-width: var(--width-medium);
}
.immutable {
  display: block;
  color: var(--blue-grey-300);
  text-transform: initial;
  margin-right: 20px;
  font-style: italic;
  margin-top: 4px;
}
</style>
