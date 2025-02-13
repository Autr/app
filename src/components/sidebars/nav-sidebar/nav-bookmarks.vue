<template>
  <div class="nav-bookmarks">
    <nav v-if="bookmarks && bookmarks.length > 0">
      <ul>
        <li v-for="bookmark in bookmarks" :key="bookmark.id" class="bookmark">
          <router-link class="no-wrap" :to="`/bookmarks/${bookmark.collection}/${bookmark.id}`">
            <v-icon name="bookmark_outline" class="icon" />
            {{ bookmark.title }}
          </router-link>
          <button
            v-if="isUserAdmin || bookmark.user === userId"
            v-tooltip="$t('delete_bookmark')"
            @click="
              confirmRemove = true;
              toBeDeletedBookmark = bookmark.id;
            "
          >
            <v-icon name="delete_outline" />
          </button>
        </li>
      </ul>
    </nav>
    <portal v-if="confirmRemove" to="modal">
      <v-confirm
        :message="$t('delete_bookmark_body')"
        @cancel="confirmRemove = false"
        @confirm="deleteBookmark"
      />
    </portal>
  </div>
</template>

<script>
export default {
  name: "NavBookmarks",
  props: {
    bookmarks: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      confirmRemove: false,
      toBeDeletedBookmark: null
    };
  },
  computed: {
    isUserAdmin() {
      return this.$store.state.currentUser.admin;
    },
    userId() {
      return this.$store.state.currentUser.id;
    }
  },
  methods: {
    deleteBookmark() {
      this.$store.dispatch("deleteBookmark", this.toBeDeletedBookmark);
      this.confirmRemove = false;
      this.toBeDeletedBookmark = null;
    }
  }
};
</script>

<style lang="scss" scoped>
h3 {
  margin-bottom: 5px;
  margin-top: 15px;
}

.icon {
  width: 20px;
  height: 22px;
  margin-right: 15px;
  color: var(--sidebar-text-color);
  fill: var(--sidebar-text-color);

  /* Forces left-alignment of material-icons */
  display: inline-flex;
  justify-content: flex-end;
  align-items: center;
  vertical-align: -7px;
}

i,
svg {
  transition: var(--fast) var(--transition);
}

.bookmark button:first-child:hover,
.user-menu button:hover {
  background-color: var(--sidebar-background-color-alt);
  border-radius: var(--border-radius);

  i,
  svg {
    color: var(--sidebar-text-color);
    fill: var(--sidebar-text-color);
  }
}

ul {
  list-style: none;
  padding: 0;
}

nav > ul > li > * {
  padding: 8px 0 8px 10px;
  margin: 2px 0;
}

.bookmark {
  display: flex;
  align-items: center;

  > * {
    display: block;
  }

  & a {
    flex-grow: 1;
    text-align: left;
    text-decoration: none;

    &:hover {
      background-color: var(--sidebar-background-color-alt);
      border-radius: var(--border-radius);
    }
  }

  & button {
    opacity: 0;
    transition: opacity var(--fast) var(--transition);
    display: flex;

    i {
      vertical-align: baseline;
      color: var(--sidebar-text-color);
    }

    &:hover i {
      color: var(--danger);
    }
  }

  &:hover button:last-child {
    opacity: 1;
  }
}
</style>
