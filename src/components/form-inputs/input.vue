<template>
  <div :class="{ 'icon-left': iconLeft, 'icon-right': iconRight }" class="v-input">
    <!-- Far from ideal, but it does the trick -->

    <input
      v-if="mask"
      :id="id"
      ref="input"
      v-mask="mask"
      :class="{ charactercount }"
      :type="type"
      :autocomplete="autocomplete"
      :max="max"
      :maxlength="maxlength"
      :min="min"
      :minlength="minlength"
      :name="name"
      :pattern="pattern"
      :placeholder="placeholder"
      :required="required"
      :readonly="readonly || disabled"
      :spellcheck="spellcheck"
      :value="value"
      :step="step"
      @keyup="$emit('keyup', $event)"
      @keydown="$emit('keydown', $event)"
      @input="$emit('input', $event.target.value)"
    />

    <input
      v-else
      :id="id"
      ref="input"
      :class="{ charactercount }"
      :type="type"
      :autocomplete="autocomplete"
      :max="max"
      :maxlength="maxlength"
      :min="min"
      :minlength="minlength"
      :name="name"
      :pattern="pattern"
      :placeholder="placeholder"
      :required="required"
      :readonly="readonly || disabled"
      :spellcheck="spellcheck"
      :value="value"
      :step="step"
      @keyup="$emit('keyup', $event)"
      @keydown="$emit('keydown', $event)"
      @input="$emit('input', $event.target.value)"
    />

    <v-icon v-if="iconLeft" v-tooltip="iconLeftTooltip" :name="iconLeft" :color="iconLeftColor" />
    <v-icon
      v-if="iconRight"
      v-tooltip="iconRightTooltip"
      :name="iconRight"
      :color="iconRightColor"
    />

    <span v-if="charactercount">{{ charsRemaining }}</span>
  </div>
</template>

<script>
export default {
  name: "VInput",
  props: {
    type: {
      type: String,
      default: "text"
    },
    autocomplete: {
      type: String,
      default: "on"
    },
    autofocus: {
      type: Boolean,
      default: false
    },
    max: {
      type: [Number, Boolean, String],
      default: null
    },
    maxlength: {
      type: [Number, Boolean, String],
      default: null
    },
    min: {
      type: [Number, Boolean, String],
      default: null
    },
    minlength: {
      type: [Number, Boolean, String],
      default: null
    },
    name: {
      type: String,
      default: ""
    },
    pattern: {
      type: String,
      default: ".*"
    },
    placeholder: {
      type: String,
      default: ""
    },
    readonly: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    },
    spellcheck: {
      type: Boolean,
      default: true
    },
    id: {
      type: String,
      default: ""
    },
    value: {
      type: null,
      default: ""
    },
    step: {
      type: [String, Number],
      default: 1
    },
    iconLeft: {
      type: String,
      default: ""
    },
    iconLeftColor: {
      type: String,
      default: null
    },
    iconLeftTooltip: {
      type: String,
      default: ""
    },
    iconRight: {
      type: String,
      default: ""
    },
    iconRightColor: {
      type: String,
      default: null
    },
    iconRightTooltip: {
      type: String,
      default: ""
    },
    valid: {
      type: Boolean,
      default: true
    },
    charactercount: {
      type: Boolean,
      default: false
    },
    mask: {
      type: [String, Array, Boolean],
      default: null
    }
  },
  computed: {
    charsRemaining() {
      if (!this.maxlength) return null;
      return this.maxlength - this.value.length;
    }
  },
  mounted() {
    if (this.autofocus) {
      this.$refs.input.focus();
    }
  }
};
</script>

<style lang="scss" scoped>
.v-input {
  position: relative;

  input {
    width: 100%;
    border: var(--input-border-width) solid var(--input-border-color);
    border-radius: var(--border-radius);
    color: var(--input-text-color);
    background-color: var(--input-background-color);
    padding: 10px;
    font-size: 1rem;
    line-height: 1.5;
    text-transform: none;
    transition: var(--fast) var(--transition);
    transition-property: color, border-color, padding;
    height: var(--input-height);

    &[type="date"] {
      -webkit-appearance: none;
    }
    &[type="date"]::-webkit-inner-spin-button {
      -webkit-appearance: none;
      display: none;
    }
    &::-webkit-clear-button {
      display: none; /* Hide the button */
      -webkit-appearance: none; /* turn off default browser styling */
    }

    &::placeholder {
      color: var(--input-placeholder-color);
    }

    &:hover:not(:read-only) {
      transition: none;
      border-color: var(--input-border-color-hover);
    }

    &:focus:not(:read-only) {
      border-color: var(--input-border-color-focus);
      outline: 0;
    }

    &:-webkit-autofill {
      box-shadow: inset 0 0 0 1000px var(--white) !important;
      color: var(--input-text-color) !important;
      -webkit-text-fill-color: var(--input-text-color) !important;
    }

    &:-webkit-autofill,
    &:-webkit-autofill:hover,
    &:-webkit-autofill:focus {
      border: var(--input-border-width) solid var(--input-border-color);
      background-color: var(--white);
      box-shadow: inset 0 0 0 2000px var(--white);
    }

    &:read-only {
      background-color: var(--input-background-color-disabled);
      border-color: var(--input-border-color);
      cursor: not-allowed;
      outline: none;
      &:focus {
        color: var(--blue-grey-400);
        border-color: var(--input-border-color);
      }
    }
  }

  span {
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    opacity: 0;
    transition: var(--fast) var(--transition);
    color: var(--input-icon-color);
    padding-left: 4px;
    background-color: var(--input-background-color);
  }

  input.charactercount:focus {
    padding-right: 30px;
  }

  input:focus + span {
    opacity: 1;
  }

  &.icon-left input {
    padding-left: 38px;
  }

  &.icon-right input {
    padding-right: 38px;
  }

  &.icon-left i,
  &.icon-right i {
    position: absolute;
    top: 50%;
    color: var(--input-icon-color);
    transform: translateY(-50%);
    font-size: 24px;

    &.accent {
      color: var(--input-background-color-active);
    }

    &.success {
      color: var(--success);
    }

    &.warning {
      color: var(--warning);
    }

    &.danger {
      color: var(--danger);
    }
  }

  &.icon-left i:first-of-type {
    left: 10px;
  }

  &.icon-right i:last-of-type {
    right: 10px;
  }
}
</style>
