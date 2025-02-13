<template>
  <transition name="fade">
    <div class="login" :class="{ loading }">
      <v-install v-if="installing" :saving="saving" @install="install" />

      <form v-else @submit.prevent="processForm">
        <img class="logo" alt="" src="../assets/logo-dark.svg" />

        <h1 v-if="loading">{{ loggedIn ? $t("fetching_data") : $t("signing_in") }}</h1>
        <h1 v-else-if="notInstalled">{{ $t("welcome") }}</h1>
        <h1 v-else>{{ resetMode ? $t("reset_password") : $t("sign_in") }}</h1>

        <label class="project-switcher">
          <select
            v-if="Object.keys(urls).length > 1 || allowOther"
            id="selectedUrl"
            v-model="selectedUrl"
            :disabled="loading"
            name="selectedUrl"
          >
            <option
              v-for="(name, u) in urls"
              :key="u"
              :value="u"
              :checked="u === url || u === url + '/'"
            >
              {{ name }}
            </option>
            <option v-if="allowOther" value="other">{{ $t("other") }}</option>
          </select>
          {{ $t("to") }}
          <span>
            {{ urls[selectedUrl] || $t("choose_project") }}
            <v-icon
              v-if="Object.keys(urls).length > 1 || allowOther"
              size="18"
              class="icon"
              name="arrow_drop_down"
            />
          </span>
        </label>

        <div v-if="selectOther" class="material-input">
          <input
            id="url"
            v-model="url"
            :disabled="loading"
            :class="{ 'has-value': url && url.length > 0 }"
            type="url"
            name="url"
          />
          <label for="url">{{ $t("api_url") }}</label>
        </div>

        <template v-if="notInstalled">
          <p class="install-copy">{{ $t("install_copy") }}</p>
          <button type="button" class="style-btn" @click="installing = true">
            {{ $t("install") }}
          </button>
        </template>

        <template v-else>
          <div v-if="!missingOTP" class="material-input">
            <input
              id="email"
              v-model="email"
              autocomplete="email"
              :disabled="loading || exists === false"
              :class="{ 'has-value': email && email.length > 0 }"
              type="text"
              name="email"
            />
            <label for="email">{{ $t("email") }}</label>
          </div>

          <div v-if="!resetMode && !missingOTP" class="material-input">
            <input
              id="password"
              v-model="password"
              autocomplete="current-password"
              :disabled="loading || exists === false"
              :class="{ 'has-value': password && password.length > 0 }"
              type="password"
              name="password"
            />
            <label for="password">{{ $t("password") }}</label>
          </div>

          <div v-if="missingOTP" class="material-input">
            <input
              id="otp"
              v-model="otp"
              :disabled="loading || exists === false"
              :class="{ 'has-value': otp && otp.length > 0 }"
              type="text"
              name="otp"
            />
            <label for="otp">{{ $t("otp") }}</label>
          </div>

          <div class="buttons">
            <button
              v-if="!missingOTP"
              type="button"
              class="forgot"
              @click.prevent="resetMode = !resetMode"
            >
              {{ resetMode ? $t("sign_in") : $t("forgot_password") }}
            </button>

            <button class="style-btn" type="submit" :disabled="disabled || loading">
              {{ resetMode ? $t("reset_password") : $t("sign_in") }}
            </button>
          </div>

          <transition-group name="list" tag="div" class="stack">
            <v-spinner
              v-if="checkingExistence || gettingThirdPartyAuthProviders"
              key="spinner"
              class="spinner"
              :size="18"
              :line-size="2"
              line-fg-color="var(--blue-grey-400)"
              line-bg-color="var(--blue-grey-200)"
            />

            <span
              v-else-if="error || SSOerror"
              key="error"
              class="notice"
              :class="errorType"
              @click="error = SSOerror = null"
            >
              <v-icon :name="errorType" />
              {{ errorMessage }}
            </span>

            <v-icon
              v-else-if="thirdPartyAuthProviders && !thirdPartyAuthProviders.length"
              key="lock"
              :name="loggedIn ? 'lock_open' : 'lock_outline'"
              class="lock"
            />

            <ul v-else key="third-party-auth" class="third-party-auth">
              <li v-for="provider in thirdPartyAuthProviders" :key="provider.name">
                <a
                  v-tooltip.bottom="$helpers.formatTitle(provider.name)"
                  :href="url + 'auth/sso/' + provider.name"
                >
                  <img :alt="provider.name" :src="provider.icon" />
                </a>
              </li>
            </ul>
          </transition-group>
        </template>
      </form>

      <small v-tooltip="{ classes: ['inverted'], content: version }" class="style-4">
        {{ $t("powered_by_directus") }}
      </small>
    </div>
  </transition>
</template>

<script>
import sdk from "@directus/sdk-js";
import { version } from "../../package.json";

import VInstall from "../components/install.vue";

export default {
  name: "Login",
  components: {
    VInstall
  },
  data() {
    return {
      selectedUrl: this.$store.state.auth.url + "/" + this.$store.state.auth.env + "/",

      url: null,
      email: null,
      password: null,
      otp: null,

      loading: false,
      loggedIn: false,
      error: null,
      SSOerror: null,

      exists: null,
      checkingExistence: false,
      thirdPartyAuthProviders: null,
      gettingThirdPartyAuthProviders: false,

      resetMode: false,

      installing: false,
      notInstalled: false,
      saving: false,

      missingOTP: false
    };
  },
  computed: {
    urls() {
      if (!window.__DirectusConfig__) return;
      return _.mapKeys(window.__DirectusConfig__.api, (val, key) =>
        key.endsWith("/") === false ? key + "/" : key
      );
    },
    allowOther() {
      if (!window.__DirectusConfig__) return;
      return window.__DirectusConfig__.allowOtherAPI;
    },
    version() {
      return `${this.$t("version")} v${version}`;
    },
    storeError() {
      return this.$store.state.auth.error;
    },
    errorType() {
      if (!this.error && !this.SSOerror) return;

      const errorCode = (this.error && this.error.code) || this.SSOerror;

      if (+errorCode >= 100 && +errorCode < 200) {
        if (+errorCode === 101 || +errorCode === 102) {
          return null;
        }

        return "warning";
      }
      return "error";
    },
    errorMessage() {
      if (!this.error && !this.SSOerror) return;

      const errorCode = (this.error && this.error.code) || this.SSOerror;

      if (this.localeMessages.errors && this.localeMessages.errors[errorCode] != null) {
        return this.$t(`errors[${errorCode}]`);
      }

      return this.$t("error_unknown");
    },
    localeMessages() {
      return this.$i18n.getLocaleMessage(this.$i18n.locale);
    },
    disabled() {
      if (this.resetMode) {
        if (this.email === null) return true;
        return this.email.length === 0;
      }

      if (this.email === null || this.password === null) return true;

      if (this.missingOTP) {
        if (this.otp === null) return true;
        else if (this.otp.length === 0) return true;
      }

      return this.email.length === 0 || this.password.length === 0;
    },
    selectOther() {
      return this.selectedUrl === "other";
    }
  },
  watch: {
    url() {
      this.exists = null;
      this.checkUrl();
    },
    selectedUrl(url) {
      if (url === "other") return;
      return (this.url = url);
    },
    exists(newVal) {
      if (newVal === true) {
        this.getThirdPartyAuthProviders();
      } else if (newVal === false) {
        this.error = { code: -1 };
      }
    },
    $route() {
      this.trySSOLogin();
    },
    storeError(error) {
      this.error = error;
    }
  },

  created() {
    this.checkUrl = _.debounce(this.checkUrl, 300);

    if (this.url) {
      this.checkUrl();
    }

    let lastUsedURL = this.$store.state.auth.url
      ? `${this.$store.state.auth.url}/${this.$store.state.auth.project}/`
      : Object.keys(window.__DirectusConfig__.api)[0];

    // Check if the last used URL is still a valid option before using it
    if (Object.keys(window.__DirectusConfig__.api).includes(lastUsedURL) === false) {
      lastUsedURL = null;
    }

    this.url = lastUsedURL || Object.keys(window.__DirectusConfig__.api)[0] || "";
    this.selectedUrl = this.url;

    if (this.url.endsWith("/") === false) this.url = this.url + "/";

    this.trySSOLogin();
  },
  beforeRouteEnter(to, from, next) {
    if (to.query.project) {
      return next(vm => {
        vm.selectedUrl = "other";
        vm.url = atob(to.query.project);
      });
    }
    return next();
  },
  methods: {
    processForm() {
      if (this.resetMode) {
        this.loading = true;

        this.$axios
          .post(this.url + "auth/password/request", {
            email: this.email
          })
          .then(() => {
            this.loading = false;
            this.$notify({
              title: this.$t("password_reset_sent", { email: this.email }),
              color: "green",
              iconMain: "check"
            });
          })
          .catch(error => {
            this.error = error;
            this.loading = false;
          });
      } else {
        const credentials = {
          url: this.url,
          email: this.email,
          password: this.password,
          otp: this.otp
        };

        this.loading = true;

        this.$store
          .dispatch("login", credentials)
          .then(() => {
            this.loggedIn = true;
            this.enterApp();
          })
          .catch(error => {
            if (error.code === 111) {
              this.missingOTP = true;
              this.error = null;
            } else if (error.code === 113) {
              this.$api.logout();
              return this.$router.push({
                path: "/2fa-activation",
                query: { temp_token: error.token }
              });
            }
            this.loading = false;
          });
      }
    },
    enterApp() {
      if (this.$route.query.redirect) {
        return this.$router.push(this.$route.query.redirect);
      }

      this.$api
        .getMe({ fields: "last_page" })
        .then(res => res.data.last_page)
        .then(lastPage => {
          if (lastPage == null || lastPage == "/logout" || lastPage == "/login") {
            lastPage = "/";
          }
          this.$router.push(lastPage);
        })
        .catch(error => {
          this.loading = false;
          this.$events.emit("error", {
            notify: this.$t("something_went_wrong_body"),
            error
          });
          this.$router.push("/");
        });
    },
    checkUrl() {
      if (!this.url) return;

      this.checkingExistence = true;
      this.exists = null;
      this.thirdPartyAuthProviders = null;
      this.error = null;
      this.notInstalled = false;

      const parts = this.url.split("/");
      parts.pop() || parts.pop();
      const newUrl = parts.join("/");

      this.$axios
        .get(newUrl + "/server/ping")
        .then(() => {
          this.exists = true;
          this.checkingExistence = false;
          this.error = null;
        })
        .catch(() => {
          this.exists = false;
          this.notInstalled = false;
          this.checkingExistence = false;
        });
    },
    getThirdPartyAuthProviders() {
      this.gettingThirdPartyAuthProviders = true;
      this.thirdPartyAuthProviders = null;

      const scopedAPI = new sdk();

      const parts = this.url.split("/");
      const env = parts.pop() || parts.pop();
      const newUrl = parts.join("/");

      scopedAPI.env = env;
      scopedAPI.url = newUrl;

      scopedAPI
        .getThirdPartyAuthProviders()
        .then(res => res.data)
        .then(thirdPartyAuthProviders => {
          this.thirdPartyAuthProviders = thirdPartyAuthProviders;
          this.gettingThirdPartyAuthProviders = false;
        })
        .catch(error => {
          if (error.code === 14) {
            this.notInstalled = true;
            this.exists = true;
          }

          this.gettingThirdPartyAuthProviders = false;
        });
    },
    trySSOLogin() {
      const queryParams = new URLSearchParams(window.location.search);
      if (queryParams.get("error")) {
        this.SSOerror = queryParams.get("code");

        const uri = window.location.toString();
        if (uri.indexOf("?") > 0) {
          const clean_uri = uri.substring(0, uri.indexOf("?"));
          window.history.replaceState({}, document.title, clean_uri);
        }

        return null;
      }

      if (queryParams.get("request_token") && queryParams.get("request_token").length > 0) {
        this.clearRequestToken();
        this.loading = true;
        this.$store
          .dispatch("loginSSO", queryParams.get("request_token"))
          .then(() => this.enterApp())
          .catch(error => {
            this.$events.emit("error", {
              notify: this.$t("something_went_wrong_body"),
              error
            });
            this.loading = false;
          });
      }
    },
    install(info) {
      const parts = this.url.split("/");
      parts.pop() || parts.pop();
      const newUrl = parts.join("/");
      this.saving = true;

      this.$axios
        .post(newUrl + "/projects", info)
        .then(() => {
          this.installing = false;
          this.exists = true;
          this.notInstalled = false;
          this.$notify({
            title: this.$t("api_installed"),
            color: "green",
            iconMain: "check"
          });
          this.saving = false;
        })
        .catch(error => {
          this.$events.emit("error", {
            notify: error.response.data.error.message,
            error
          });

          this.saving = false;
        });
    },
    clearRequestToken() {
      const url = window.location.href;
      const newUrl = url
        .replace(new RegExp("[?&]request_token=[^&#]*(#.*)?$"), "$1")
        .replace(new RegExp("([?&])request_token=[^&]*&"), "$1");

      history.replaceState({}, "Login | Directus", newUrl);
    }
  }
};
</script>

<style lang="scss" scoped>
.login {
  background-color: var(--blue-grey-900);
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

form {
  background-color: var(--white);
  border-radius: var(--border-radius);
  max-width: 340px;
  width: 100%;
  position: relative;
  top: -2vh;
  padding: 40px 40px 30px;
  box-shadow: var(--box-shadow);
}

.logo {
  height: 40px;
  margin-bottom: 40px;
  pointer-events: none;
  user-select: none;
}

label {
  position: relative;
  text-transform: none;
  font-size: 16px;
  color: var(--blue-grey-300);
  font-weight: 400;
}

h1 {
  color: var(--blue-grey-800);
  font-size: 36px;
  line-height: 1.4;
  font-weight: 300;
}

select {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0;
  z-index: +1;
  left: 0;
  top: 0;
  font-size: 16px;
  cursor: pointer;
}

.project-switcher {
  margin-bottom: 50px;

  span {
    transition: color var(--fast) var(--transition);
  }

  select:hover + span,
  select:focus + span {
    transition: none;
    cursor: pointer;
    color: var(--blue-grey-800);
  }

  .icon {
    width: 10px;
    height: 10px;
    vertical-align: top;
  }
}

.material-input {
  position: relative;
  width: 100%;
  margin: 40px 0 20px;

  input {
    border: 0;
    font-size: 16px;
    border-bottom: 2px solid var(--blue-grey-200);
    width: 100%;
    padding: 10px 0;
    color: var(--blue-grey-400);
    transition: border-color var(--fast) var(--transition);
    border-radius: 0;

    &:-webkit-autofill {
      color: var(--blue-grey-400) !important;
      -webkit-text-fill-color: var(--blue-grey-400);
      -webkit-box-shadow: 0 0 0px 1000px var(--white) inset;
    }

    &:hover:not([disabled]) {
      transition: none;
      border-color: var(--blue-grey-400);
      &:focus {
        border-color: var(--blue-grey-800);
      }
    }

    &[disabled] {
      cursor: not-allowed;
    }

    &:focus {
      outline: 0;
      border-color: var(--blue-grey-800);
      color: var(--blue-grey-600);

      &:-webkit-autofill {
        color: var(--blue-grey-800) !important;
        -webkit-text-fill-color: var(--blue-grey-800);
        -webkit-box-shadow: 0 0 0px 1000px var(--white) inset;
      }
    }
  }

  label {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    font-size: 16px;
    font-weight: 400;
    transform-origin: left;
    transition: all var(--fast) var(--transition);
  }

  input:focus,
  input.has-value {
    & + label {
      transform: scale(0.8);
      top: -10px;
    }
  }

  input:-webkit-autofill {
    /* Fixes FireFox: If any part of a selector is invalid, it invalidates the whole selector */
    & + label {
      transform: scale(0.8);
      top: -10px;
    }
  }
}

.forgot {
  float: right;
  display: block;
  text-decoration: none;
  margin-bottom: 50px;
  transition: color var(--fast) var(--transition-out);
  color: var(--blue-grey-300);

  &:hover,
  .user-is-tabbing &:focus {
    outline: 0;
    transition: none;
    color: var(--blue-grey-900);
  }
}

button.style-btn {
  font-size: 16px;
  font-weight: 500;
  background-color: var(--blue-grey-800);
  width: 100%;
  display: block;
  padding: 12px 0;
  border-radius: var(--border-radius);
  transition: background-color var(--fast) var(--transition);
  position: relative;
  top: 0;

  &[disabled] {
    background-color: var(--blue-grey-200);
    cursor: not-allowed;
  }

  &:not([disabled]) {
    &:hover {
      background-color: var(--blue-grey-900);
    }
  }
}

.lock {
  display: block;
  margin: 0 auto;
  width: max-content;
  pointer-events: none;
  user-select: none;
  color: var(--blue-grey-200);
}

small {
  position: absolute;
  bottom: 20px;
  text-align: center;
  cursor: help;
  left: 0;
  right: 0;
  margin-left: auto;
  margin-right: auto;
}

.third-party-auth {
  padding: 0;
  list-style: none;
  display: flex;
  justify-content: space-around;

  a {
    display: block;
  }

  img {
    width: 21px;
    height: 21px;
    object-fit: contain;
  }
}

.list-enter-active {
  transition: var(--fast) var(--transition-in);
}

.list-leave-active {
  transition: var(--fast) var(--transition-out);
}

.list-enter,
.list-leave-to {
  opacity: 0;
}

.stack {
  position: relative;
  width: 100%;
  padding: 30px 0 20px;
  text-align: center;

  > * {
    position: absolute;
    left: 0;
    right: 0;
    margin: 0 auto;
  }

  .lock {
    transform: translateY(-3px);
  }
}

.close {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 5;
  cursor: pointer;
  opacity: 0;
}

.fade-enter-active {
  opacity: 1;
  transition: var(--slow) var(--transition-in);

  form {
    transition: transform var(--slow) var(--transition-in);
  }
}

.fade-leave-active {
  transition: var(--slow) var(--transition-out);
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;

  form {
    transition: transform var(--slow) var(--transition-out);
  }
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.buttons {
  max-height: 105px;
  transition: max-height var(--slow) var(--transition-in);
}

.loading {
  pointer-events: none;
  user-select: none;

  .material-input {
    input {
      transition: var(--fast) var(--transition);
      border-color: var(--white);
    }
  }

  .buttons {
    max-height: 0px;
    transition: var(--slow) var(--transition);
    opacity: 0;
    overflow: hidden;
  }

  .project-switcher i {
    display: none;
  }
}

@media (max-height: 650px) {
  form {
    padding: 30px 25px 20px;
  }

  .logo {
    margin-bottom: 20px;
  }

  .forgot {
    margin-bottom: 20px;
  }

  .stack {
    padding: 20px 0;
  }
}

.notice {
  text-align: center;
  cursor: pointer;
  &.error {
    color: var(--danger);
  }
}

.install-copy {
  margin-bottom: 80px;
  max-width: 400px;
  font-size: 16px;
  color: var(--blue-grey-300);
  line-height: 26px;
  font-weight: 400;
}
</style>
