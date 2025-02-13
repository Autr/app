<template>
  <div class="layout-cards" @scroll="onScroll">
    <div class="toolbar">
      <p>{{ $t("sort_by") }}</p>

      <div class="sort-select">
        <select :value="sortedOn" @input="setSort($event.target.value)">
          <option v-for="(fieldInfo, name) in sortableFields" :key="name" :value="name">
            {{ $helpers.formatTitle(name) }}
          </option>
        </select>
        <v-icon class="icon" name="arrow_drop_down" />
      </div>

      <div class="sort-select">
        <select :value="sortDirection" @input="setSortDirection($event.target.value)">
          <option value="asc">{{ $t("ASC") }}</option>
          <option value="desc">{{ $t("DESC") }}</option>
        </select>
        <v-icon class="icon" name="arrow_drop_down" />
      </div>
    </div>

    <div class="cards" :class="{ loading: loading }">
      <v-card
        v-for="(item, index) in items"
        :key="item.id"
        :to="item[link]"
        :icon="emptySrc(item) ? viewOptions.icon || 'photo' : null"
        :opacity="emptySrc(item) ? 'half' : null"
        :src="src(item)"
        :selected="selection.includes(item.id)"
        :selection-mode="selection.length > 0"
        @select="select(item.id)"
      >
        <template slot="title">
          <v-ext-display
            :key="`card-title-${fields[title].interface}-${index}`"
            :interface-type="fields[title].interface"
            :name="title"
            :collection="collection"
            :type="fields[title].type"
            :options="fields[title].options"
            :value="item[title]"
            :relation="fields[title].relation"
          />
        </template>
        <template v-if="subtitle" slot="subtitle">
          <v-ext-display
            :key="`card-subtitle-${fields[subtitle].interface}-${index}`"
            :interface-type="fields[subtitle].interface"
            :name="subtitle"
            :collection="collection"
            :type="fields[subtitle].type"
            :options="fields[subtitle].options"
            :value="item[subtitle]"
            :relation="fields[subtitle].relation"
          />
        </template>
        <template v-if="content" slot="content">
          <v-ext-display
            :key="`card-content-${fields[content].interface}-${index}`"
            :interface-type="fields[content].interface"
            :name="content"
            :collection="collection"
            :type="fields[content].type"
            :options="fields[content].options"
            :value="item[content]"
            :relation="fields[content].relation"
          />
        </template>
      </v-card>
      <v-card
        v-if="lazyLoading"
        icon="hourglass_empty"
        opacity="half"
        :title="$t('loading_more')"
        :subtitle="$t('one_moment')"
      ></v-card>
    </div>

    <div v-if="loading" class="layout-loading">
      <v-spinner />
    </div>
  </div>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/layout";

export default {
  name: "LayoutCards",
  mixins: [mixin],
  computed: {
    title() {
      return this.viewOptions.title || this.primaryKeyField;
    },
    subtitle() {
      return this.viewOptions.subtitle;
    },
    content() {
      return this.viewOptions.content;
    },
    sortableFields() {
      return _.pickBy(this.fields, field => field.datatype);
    },
    sortedOn() {
      let fieldName;
      const sortableFieldNames = Object.keys(this.sortableFields);
      const viewQuerySort = this.viewQuery.sort;
      if (
        sortableFieldNames &&
        viewQuerySort &&
        sortableFieldNames.some(sortableFieldName => sortableFieldName === viewQuerySort)
      ) {
        fieldName = viewQuerySort;
      } else if (sortableFieldNames && sortableFieldNames.length > 0) {
        // If the user didn't sort, default to the first field
        fieldName = sortableFieldNames[0];
      } else {
        return null;
      }

      // If the sort viewQuery was already descending, remove the - so we don't
      // run into server errors with double direction characters
      if (fieldName.startsWith("-")) fieldName = fieldName.substring(1);

      return fieldName;
    },
    sortDirection() {
      if (!this.viewQuery.sort) return "asc";

      if (this.viewQuery.sort.substring(0, 1) === "-") return "desc";

      return "asc";
    }
  },
  methods: {
    src(item) {
      const srcField = this.viewOptions.src || null;

      if (srcField) {
        if (this.fields[srcField] && this.fields[srcField].type.toLowerCase() === "file") {
          return (
            item[srcField] &&
            item[srcField].data &&
            item[srcField].data.thumbnails &&
            item[srcField].data.thumbnails[0] &&
            item[srcField].data.thumbnails[0].url
          );
        }

        if (srcField === "data" && this.fields[srcField].collection === "directus_files") {
          return (
            item[srcField] &&
            item[srcField].thumbnails &&
            item[srcField].thumbnails[0] &&
            item[srcField].thumbnails[0].url
          );
        }

        return item[srcField] || null;
      }

      return null;
    },
    emptySrc(item) {
      return this.viewOptions.src != null && this.src(item) === null;
    },
    onScroll(event) {
      const { scrollHeight, clientHeight, scrollTop } = event.srcElement;
      const totalScroll = scrollHeight - clientHeight;
      const delta = totalScroll - scrollTop;
      if (delta <= 500) this.$emit("next-page");
      this.scrolled = scrollTop > 0;
    },
    select(id) {
      let newSelection;

      if (this.selection.includes(id)) {
        newSelection = this.selection.filter(selectedID => selectedID !== id);
      } else {
        newSelection = [...this.selection, id];
      }

      this.$emit("select", newSelection);
    },
    setSort(fieldName) {
      this.$emit("query", {
        sort: fieldName
      });
    },
    setSortDirection(direction) {
      this.$emit("query", {
        sort: (direction === "desc" ? "-" : "") + this.sortedOn
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.layout-cards {
  overflow: auto;
  height: 100%;
  max-height: calc(100vh - var(--header-height));
}

.toolbar {
  background-color: var(--page-background-color);
  width: 100%;
  position: sticky;
  top: 0;
  z-index: +1;
  height: var(--header-height);
  padding: 28px var(--page-padding);
  display: flex;
  align-items: center;
  justify-content: flex-end;
  border-bottom: 2px solid var(--sidebar-background-color);
}

.sort-select {
  position: relative;
  display: flex;
  align-items: center;

  margin: 0 10px;

  &:last-of-type {
    margin-right: 0;
  }

  select {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    vertical-align: middle;
    background-color: var(--input-background-color-alt);
    border-radius: var(--border-radius);
    border: 0;
    overflow: hidden;
    padding: 5px;
    padding-right: 15px;
    cursor: pointer;
    outline: 0;
  }

  .icon {
    pointer-events: none;
    position: absolute;
    right: 0;
    top: 50%;
    transform: translateY(-50%);
  }
}

.cards {
  padding: var(--page-padding);
  padding-bottom: var(--page-padding-bottom);
  display: grid;
  grid-template-columns: repeat(auto-fill, 136px);
  grid-gap: 30px 20px;
  justify-content: space-between;
  width: 100%;

  &.loading {
    opacity: 0.5;
  }
}

.layout-loading {
  padding: 24px 0;
}
</style>
