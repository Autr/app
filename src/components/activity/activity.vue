<template>
  <div class="v-activity">
    <form
      v-show="commentPermission !== 'none' && commentPermission !== 'read'"
      class="new-comment"
      @submit.prevent="postComment"
    >
      <v-textarea
        v-model="comment"
        class="textarea"
        :rows="5"
        required
        :placeholder="$t('leave_comment')"
      />

      <button type="submit" :disabled="comment.trim().length === 0">
        {{ $t("submit") }}
      </button>
    </form>

    <article
      v-for="(activity, index) in activityWithChanges"
      :key="activity.id"
      class="activity-item"
    >
      <span
        v-tooltip="$helpers.formatTitle(activity.action)"
        :class="activity.action"
        class="indicator"
      />

      <div class="content">
        <details v-if="activity.action !== 'external' && activity.changes && activity.name">
          <summary class="title">
            <span class="name">{{ activity.name }}</span>
            <v-time-ago
              v-if="activity.date"
              v-tooltip="{
                content: $d(activity.date, 'long'),
                delay: { show: 1500, hide: 100 }
              }"
              :datetime="activity.date"
              :locale="$i18n.locale"
              class="date"
            />
            <v-icon v-tooltip="'Revision Details'" class="chevron" name="chevron_left" size="18" />
          </summary>
          <div v-if="activity.changes">
            <v-diff :changes="activity.changes" />
            <button
              v-if="index !== 0"
              v-tooltip="$t('revert')"
              class="revert"
              @click="$emit('revert', activity)"
            >
              <v-icon name="restore" />
            </button>
          </div>
        </details>
        <div v-else-if="activity.name" class="title">
          <span class="name">{{ activity.name }}</span>
          <v-time-ago
            v-if="activity.date"
            v-tooltip="{
              content: $d(activity.date, 'long'),
              delay: { show: 1500, hide: 100 }
            }"
            :datetime="activity.date"
            :locale="$i18n.locale"
            class="date"
          />
        </div>
        <p
          v-if="activity.htmlcomment"
          :class="{
            comment: activity.action && activity.action.toLowerCase() === 'comment'
          }"
          v-html="activity.htmlcomment"
        ></p>
      </div>
    </article>
  </div>
</template>

<script>
import VDiff from "./diff.vue";
import { diff } from "deep-object-diff";

export default {
  name: "VActivity",
  components: {
    VDiff
  },
  props: {
    activity: {
      type: Array,
      required: true
    },
    revisions: {
      type: Object,
      required: true
    },
    loading: {
      type: Boolean,
      default: false
    },
    commentPermission: {
      type: String,
      default: "none"
    }
  },
  data() {
    return {
      comment: ""
    };
  },
  computed: {
    activityWithChanges() {
      const activityWithChanges = this.activity.map((activity, i) => ({
        ...activity,
        changes: this.getChanges(activity.id, i),
        revision: this.revisions[activity.id]
      }));

      const lastItem = activityWithChanges && activityWithChanges[activityWithChanges.length - 1];

      if (!lastItem) {
        activityWithChanges.push({
          action: "external",
          comment: this.$t("activity_outside_directus"),
          id: -1
        });
      } else {
        const create = lastItem.action.toLowerCase() === "create";
        const upload = lastItem.action.toLowerCase() === "upload";

        if (!create && !upload) {
          activityWithChanges.push({
            action: "external",
            comment: this.$t("activity_outside_directus"),
            id: -1
          });
        }
      }

      return activityWithChanges.map(activity => ({
        ...activity,
        htmlcomment: this.$helpers.snarkdown(
          (activity.comment || "")
            // Remove headings because they're ugly basically
            .replace(/#/g, "")
            // Cleanup the comment, and escape HTML chars in order to prevent
            // XSS style problems
            .replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;")
            .replace(/"/g, "&quot;")
            .replace(/'/g, "&#039;") || ""
        )
      }));
    }
  },
  methods: {
    getChanges(activityID, index) {
      const revision = this.revisions[activityID];

      if (!revision) return null;

      let previousUpdate = null;

      for (let i = index + 1; i < this.activity.length; i++) {
        if (
          this.activity[i].action === "update" ||
          this.activity[i].action === "create" ||
          this.activity[i].action === "upload"
        ) {
          previousUpdate = this.activity[i];
          break;
        }
      }

      if (!previousUpdate) {
        if (this.activity[index].action === "create") {
          const data = revision.data;

          return _.mapValues(data, (value, field) => ({
            before: null,
            after: value,
            field: field
          }));
        }

        return null;
      }

      const previousRevision = this.revisions[previousUpdate.id];
      const previousData = (previousRevision && previousRevision.data) || {};
      const currentData = revision.data || {};
      const currentDelta = revision.delta;

      // The API will save the delta no matter if it actually changed or not
      const localDelta = diff(_.clone(previousData), _.clone(currentData));

      const hasChanged = Object.keys(localDelta).length > 0;

      if (hasChanged === false) return null;

      return _.mapValues(currentDelta, (value, field) => ({
        before: previousData[field],
        after: value,
        field
      }));
    },
    postComment() {
      this.$emit("input", this.comment);
      this.comment = "";
    }
  }
};
</script>

<style lang="scss" scoped>
.v-activity {
  position: relative;

  &::before {
    content: "";
    position: absolute;
    left: 4px;
    top: 80px;
    bottom: 8px;
    width: 2px;
    background-color: var(--input-border-color);
    z-index: -1;
  }

  .indicator {
    position: relative;
    top: 4px;
    display: inline-block;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: var(--input-border-color);
    // box-shadow: 0 0 0 2px var(--blue-grey-50);
    flex-shrink: 0;

    &.update {
      background-color: var(--blue-grey-600);
    }
    &.delete {
      background-color: var(--danger);
    }
    &.add {
      background-color: var(--success);
    }
    &.external {
      background-color: var(--input-border-color);
    }
    &.upload {
      background-color: var(--purple-500);
    }
  }

  article {
    display: flex;
    margin-bottom: 30px;
  }

  .content {
    margin-left: 10px;
    flex-grow: 1;

    .name {
      font-weight: 500;
      color: var(--heading-text-color);
    }

    .date {
      color: var(--note-text-color);
      margin-left: 8px;
    }

    .title {
      list-style-type: none;

      &::-webkit-details-marker {
        display: none;
      }
    }

    summary {
      position: relative;
      width: 224px;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      padding-right: 20px;
      cursor: pointer;
      color: var(--blue-grey-300);

      &:hover {
        .chevron {
          color: var(--blue-grey-600);
        }
      }

      .chevron {
        position: absolute;
        top: 0;
        right: 0;
        color: var(--blue-grey-200);
        transition: all var(--fast) var(--transition);
      }
    }

    > *:not(:first-child) {
      margin-top: 10px;
    }

    .revert {
      transition: all var(--fast) var(--transition);
      background-color: var(--blue-grey-200);
      border-radius: var(--border-radius);
      padding: 4px;
      margin: 14px auto;
      width: 100%;
      i {
        width: auto;
        height: auto;
        transform: translateX(0);
        background-color: inherit;
        font-size: 24px;
        color: var(--blue-grey-50);
      }
      &:hover {
        background-color: var(--blue-grey-600);
        i.material-icons {
          color: var(--white);
        }
      }
    }

    .comment {
      position: relative;
      background-color: var(--sidebar-background-color-alt);
      color: var(--sidebar-text-color);
      border-radius: var(--border-radius);
      padding: 8px 10px;
      display: inline-block;
      min-width: 36px;

      &:before {
        content: "";
        position: absolute;
        top: -6px;
        left: 10px;
        display: inline-block;
        width: 0;
        height: 0;
        border-style: solid;
        border-width: 0 8px 6px 8px;
        border-color: transparent transparent var(--sidebar-background-color-alt) transparent;
      }
      a {
        color: var(--blue-grey-800);
        text-decoration: none;
        &:hover {
          color: var(--blue-grey-900);
        }
      }
      strong {
        font-weight: 600;
      }
      code {
        font-family: "Roboto Mono";
        color: var(--blue-grey-400);
        font-weight: 600;
      }
      pre {
        font-family: "Roboto Mono";
        color: var(--blue-grey-400);
        font-weight: 600;
        background-color: var(--blue-grey-200);
        padding: 4px 6px;
        border-radius: var(--border-radius);
        margin: 4px 0;
      }
      ul,
      ol {
        margin: 4px 0;
        padding-left: 25px;
      }
      blockquote {
        font-size: 1.2em;
        font-weight: 400;
        margin: 20px 10px 20px 10px;
        border-left: 2px solid var(--blue-grey-200);
        padding-left: 10px;
        color: var(--blue-grey-900);
        line-height: 1.4em;
      }
      hr {
        margin: 20px 0;
        height: 1px;
        border: none;
        background-color: var(--blue-grey-200);
      }
    }
  }

  details[open] .chevron {
    transform: rotate(-90deg);
    transform-origin: 50% 60%;
  }
}

.new-comment {
  position: relative;
  height: var(--input-height);
  transition: height var(--slow) var(--transition);
  margin-bottom: 30px;

  .textarea {
    height: 100%;
    resize: none;
    line-height: 20px;
  }

  button {
    position: absolute;
    bottom: 8px;
    right: 12px;
    transition: var(--fast) var(--transition);
    transition-property: color, opacity;
    font-weight: var(--weight-bold);
    opacity: 0;
    color: var(--input-background-color-active);
    background-color: var(--input-background-color);
    cursor: pointer;

    &[disabled] {
      color: var(--input-border-color);
      cursor: not-allowed;
    }
  }

  &:focus,
  &:focus-within {
    height: calc(3 * var(--input-height));

    button {
      opacity: 1;
    }
  }
}
</style>

<style lang="scss">
.v-activity .content .comment {
  a {
    // color: var(--blue-grey-800);
    // text-decoration: underline;
    &:hover {
      color: var(--blue-grey-900);
    }
  }
  strong {
    font-weight: 600;
    color: var(--blue-grey-400);
  }
  code {
    font-family: "Roboto Mono";
    color: var(--blue-grey-400);
    font-weight: 600;
  }
  pre {
    font-family: "Roboto Mono";
    color: var(--blue-grey-400);
    font-weight: 600;
    background-color: var(--blue-grey-200);
    padding: 4px 6px;
    border-radius: var(--border-radius);
    margin: 10px 0;
  }
  ul,
  ol {
    margin: 10px 0;
    padding-left: 25px;
  }
  blockquote {
    font-size: 1.2em;
    font-weight: 400;
    margin: 20px 10px 20px 10px;
    border-left: 2px solid var(--blue-grey-200);
    padding-left: 10px;
    color: var(--blue-grey-900);
    line-height: 1.4em;
  }
  hr {
    margin: 20px 0;
    height: 1px;
    border: none;
    background-color: var(--blue-grey-200);
  }
}
</style>
