<template>
  <div class="v-diff">
    <div v-for="{ field, before, after } in changes" :key="field" class="change">
      <p>{{ $helpers.formatTitle(field) }}</p>
      <div class="diff">
        <div :class="{ empty: !before }" class="before">
          {{ before || "--" }}
        </div>
        <div :class="{ empty: !after }" class="after">{{ after || "--" }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "VDiff",
  props: {
    changes: {
      type: Object,
      required: true
    }
  }
};
</script>

<style lang="scss" scoped>
.change {
  width: 100%;
  margin-top: 16px;

  p {
    margin-bottom: 4px;
    // color: var(--blue-grey-300);
  }

  .diff {
    width: 100%;
    border-radius: var(--border-radius);
    overflow: hidden;

    > div {
      width: 100%;
      padding: 4px 20px 4px 4px;
      font-size: 13px;
    }
  }

  .before {
    position: relative;
    color: var(--danger);
    background-color: var(--white);
    border-bottom: 2px solid var(--blue-grey-50);
    max-height: 300px;
    overflow: auto;
    &:after {
      content: "close";
      position: absolute;
      right: 0px;
      top: 50%;
      transform: translateY(-50%);
      font-family: "Material Icons";
      font-feature-settings: "liga";
      color: var(--danger);
      display: inline-block;
      vertical-align: middle;
      margin: 0 5px;
    }
  }

  .after {
    position: relative;
    color: var(--success);
    background-color: var(--white);
    max-height: 300px;
    overflow: auto;
    &:after {
      content: "check";
      position: absolute;
      right: 0px;
      top: 50%;
      transform: translateY(-50%);
      font-family: "Material Icons";
      font-feature-settings: "liga";
      color: var(--success);
      display: inline-block;
      vertical-align: middle;
      margin: 0 5px;
    }
  }

  .empty {
    color: var(--blue-grey-200);
    background-color: var(--white);
    &:after {
      content: "block";
      position: absolute;
      right: 0px;
      top: 50%;
      transform: translateY(-50%);
      font-family: "Material Icons";
      font-feature-settings: "liga";
      color: var(--blue-grey-200);
      display: inline-block;
      vertical-align: middle;
      margin: 0 5px;
    }
  }
}
</style>
