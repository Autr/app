<template>
  <transition name="slide">
    <div class="v-modal">
      <div
        :class="actionRequired ? null : 'pointer'"
        class="modal-mask"
        @click="actionRequired ? null : $emit('close')"
      />
      <div class="modal-wrapper">
        <aside
          ref="modal"
          class="modal-container"
          :class="{ 'full-size': fullSize }"
          aria-labelledby="modal-title"
          aria-describedby="modal-description"
          role="dialog"
          @keydown.esc="actionRequired ? null : $emit('close')"
        >
          <div role="document">
            <div id="modal-description" class="screen-reader-offscreen">
              {{ $t("dialog_beginning") }}
              {{ actionRequired ? null : $t("esc_cancel") }}
            </div>
            <header v-if="title">
              <h1 id="modal-title" class="type-heading-small">{{ title }}</h1>
              <button v-if="!actionRequired" @click="$emit('close')">
                <v-icon name="close" />
              </button>
            </header>

            <div v-if="tabs" class="tabs">
              <button
                v-for="(info, id) in tabs"
                :key="id"
                :class="{ active: activeTab === id }"
                :disabled="info.disabled"
                @click="$emit('tab', id)"
              >
                {{ info.text }}
              </button>
            </div>

            <div ref="tabsBody" class="body">
              <template v-if="tabs">
                <div
                  v-for="(info, id) in tabs"
                  v-show="activeTab === id"
                  :key="`tab-${id}`"
                  class="tab"
                >
                  <slot :name="id" />
                </div>
              </template>

              <slot v-else />
            </div>
            <div class="footer" :class="{ 'action-required': actionRequired }">
              <button v-if="!actionRequired" @click="$emit('close')">
                {{ $t("cancel") }}
              </button>
              <v-button
                v-for="(button, id) in buttons"
                :key="id"
                class="confirm"
                :loading="button.loading || false"
                :disabled="button.disabled || false"
                @click="$emit(id)"
              >
                {{ button.text }}
              </v-button>
            </div>
          </div>
        </aside>
      </div>
    </div>
  </transition>
</template>

<script>
import { disableBodyScroll, enableBodyScroll } from "body-scroll-lock";

export default {
  name: "VModal",
  props: {
    actionRequired: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: null
    },
    buttons: {
      type: Object,
      default: () => ({})
    },
    tabs: {
      type: Object,
      default: null
    },
    activeTab: {
      type: String,
      default: null
    },
    fullSize: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    /*
     * Focus on the first input in the currently rendered tab on change of tab
     * Improves keyboard navigation greatly
     */
    activeTab() {
      // A timeout is needed to allow the browser to render the dom first before
      // firing the focus method
      setTimeout(() => {
        this.$refs.tabsBody.querySelector("input").focus();
      }, 0);
    }
  },
  mounted() {
    disableBodyScroll(this.$refs.modal);
  },
  beforeDestroy() {
    enableBodyScroll(this.$refs.modal);
  }
};
</script>

<style lang="scss" scoped>
.v-modal {
  display: table;
  position: fixed;
  z-index: 1500;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.modal-mask {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background-color: var(--blue-grey-900);
  opacity: 0.9;

  &.pointer {
    cursor: pointer;
  }
}

.modal-wrapper {
  position: relative;
  display: table-cell;
  vertical-align: middle;
  pointer-events: none;
  opacity: 1;
  z-index: +1;
}

.modal-container {
  position: relative;
  margin: 0 auto;
  width: 90%;
  max-width: 690px;
  background-color: var(--white);
  border-radius: var(--border-radius);
  box-shadow: var(--box-shadow);
  transition: inherit;
  pointer-events: painted;
  cursor: default;
  overflow: hidden;

  &.full-size {
    max-width: none;
    width: calc(100% - 40px);
    margin: 20px;
  }

  h1 {
    line-height: 18px; /* match close icon */
  }

  header,
  .footer {
    padding: 20px 30px;
  }

  header,
  .footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .footer.action-required {
    justify-content: flex-end;
  }

  header {
    position: sticky;
    top: 0;
    height: 64px;
    background-color: var(--blue-grey-50);
  }

  .body {
    overflow: auto;
    -webkit-overflow-scrolling: touch;
    max-height: 70vh;
    position: relative;
    background-color: var(--body-background);

    .tab {
      height: 100%;
      position: relative;
    }
  }

  .footer {
    border-top: 2px solid var(--blue-grey-50);
    height: 64px;
  }

  button:not(.confirm) {
    transition: var(--fast) var(--transition);
    color: var(--blue-grey-300);

    &:hover {
      color: var(--blue-grey-600);
    }
  }
}

.screen-reader-offscreen {
  position: absolute;
  left: -99999px;
}

.slide-enter-active,
.slide-enter-active .modal-container {
  transition: var(--slow) var(--transition-in);
}

.slide-leave-active,
.slide-leave-active .modal-container {
  transition: var(--medium) var(--transition-out);
}

.slide-enter,
.slide-leave-to {
  opacity: 0;
}

.tabs {
  display: flex;
  padding: 0;
  list-style: none;
  justify-content: center;
  border-bottom: 2px solid var(--blue-grey-50);
  position: sticky;
  top: 0;
  background-color: var(--white);
  z-index: +1;

  button {
    flex-grow: 1;
    flex-shrink: 1;
    max-width: 120px;
    flex-basis: 120px;
    height: 48px;
    position: relative;
    color: var(--blue-grey-400);

    text-decoration: none;
    text-transform: uppercase;
    font-size: 12px;
    font-weight: 500;
    position: relative;

    &:hover {
      color: var(--blue-grey-800);
    }

    &::after {
      content: "";
      display: block;
      width: 100%;
      position: absolute;
      height: 2px;
      bottom: -2px;
      background-color: var(--blue-grey-900);
      transform: scaleY(0);
      transition: transform var(--fast) var(--transition-out);
    }

    &.active {
      color: var(--blue-grey-800);

      &:hover {
        color: var(--blue-grey-900);
      }

      &::after {
        transform: scaleY(1);
        transition: transform var(--fast) var(--transition-in);
      }
    }

    &[disabled] {
      color: var(--blue-grey-200);
      cursor: not-allowed;
      &:hover {
        color: var(--blue-grey-200);
      }
    }
  }
}

.tab {
  padding: 30px;
}
</style>
