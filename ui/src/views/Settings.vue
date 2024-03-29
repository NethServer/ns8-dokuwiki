<template>
  <div class="bx--grid bx--grid--full-width">
    <div class="bx--row">
      <div class="bx--col-lg-16 page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </div>
    </div>
    <div v-if="error.getConfiguration" class="bx--row">
      <div class="bx--col">
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </div>
    </div>
    <div v-if="already_set" class="bx--row">
      <div class="bx--col">
        <NsInlineNotification
          kind="info"
          :title="$t('settings.dokuwiki_note')"
          :description="$t('settings.must_be_configured_inside_dokuwiki')"
          :showCloseButton="false"
          @click="goToDokuwikiWebapp"
          :actionLabel="$t('settings.go_to_dokuwiki')"
        />
      </div>
    </div>
    <div class="bx--row">
      <div class="bx--col-lg-16">
        <cv-tile :light="true">
          <cv-skeleton-text
            v-show="loading.getConfiguration || loading.configureModule"
            heading
            paragraph
            :line-count="15"
            width="80%"
          ></cv-skeleton-text>
          <cv-form v-show="!(loading.getConfiguration || loading.configureModule)" @submit.prevent="configureModule">
            <cv-text-input
              v-if="!already_set"
              :label="$t('settings.wiki_name')"
              v-model.trim="wikiName"
              class="mg-bottom"
              :invalid-message="$t(error.wiki_name)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="wikiName"
            >
            </cv-text-input>
            <template v-if="ldap_domain == '-'">
              <cv-text-input
                :label="$t('settings.admin_username')"
                v-model.trim="username"
                class="mg-bottom"
                :invalid-message="$t(error.username)"
                :disabled="
                  loading.getConfiguration ||
                  loading.configureModule ||
                  already_set
                "
                ref="username"
              >
              </cv-text-input>
              <cv-text-input
                :label="$t('settings.admin_password')"
                v-model.trim="password"
                type="password"
                :password-show-label="$t('settings.show_password')"
                :password-hide-label="$t('settings.hide_password')"
                class="mg-bottom"
                :invalid-message="$t(error.password)"
                :disabled="
                  loading.getConfiguration ||
                  loading.configureModule ||
                  already_set
                "
                ref="password"
              >
              </cv-text-input>
              <cv-text-input
                v-if="!already_set"
                :label="$t('settings.admin_email')"
                placeholder="admin@example.com"
                v-model.trim="email"
                class="mg-bottom"
                :invalid-message="$t(error.email)"
                :disabled="loading.getConfiguration || loading.configureModule"
                ref="email"
              >
              </cv-text-input>
              <cv-text-input
                v-if="!already_set"
                :label="$t('settings.admin_full_name')"
                v-model.trim="userFullName"
                class="mg-bottom"
                :invalid-message="$t(error.user_full_name)"
                :disabled="loading.getConfiguration || loading.configureModule"
                ref="userFullName"
              >
              </cv-text-input>
            </template>
            <cv-text-input
              :label="$t('settings.wiki_fqdn')"
              placeholder="mywiki.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            >
            </cv-text-input>
            <NsComboBox
              v-if="already_set"
              v-model.trim="ldap_domain"
              :autoFilter="true"
              :autoHighlight="true"
              :title="$t('settings.ldap_domain')"
              :label="$t('settings.choose_ldap_domain')"
              :options="ldap_domain_list"
              :userInputLabel="core.$t('common.user_input_l')"
              :acceptUserInput="false"
              :showItemType="true"
              :invalid-message="$t(error.ldap_domain)"
              :disabled="loading.getConfiguration || loading.configureModule"
              tooltipAlignment="start"
              tooltipDirection="top"
              ref="ldap_domain"
            >
              <template slot="tooltip">
                {{
                  $t("settings.choose_the_ldap_domain_to_authenticate_users")
                }}
              </template>
            </NsComboBox>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <div v-if="error.configureModule" class="bx--row">
              <div class="bx--col">
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </div>
            </div>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </div>
    </div>
  </div>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [TaskService, IconService, UtilService, QueryParamService],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      already_set: false,
      wikiName: "",
      username: "",
      password: "",
      email: "",
      userFullName: "",
      host: "",
      ldap_domain: "",
      ldap_domain_list: [],
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: false,
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        wiki_name: "",
        username: "",
        password: "",
        user_full_name: "",
        email: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
        ldap_domain: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    goToDokuwikiWebapp() {
      window.open(`https://${this.host}`, "_blank");
    },
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.core.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.wikiName = config.wiki_name;
      this.username = config.username;
      this.password = config.password;
      this.userFullName = config.user_full_name;
      this.email = config.email;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      // force to reload value after dom update
      this.$nextTick(() => {
        this.ldap_domain = config.ldap_domain;
        if (this.ldap_domain == "") {
          this.ldap_domain = "-";
        }
      });
      // for NSComboBox we need to create a temporary variable
      let ldap_domain_list_tmp = config.ldap_domain_list;
      ldap_domain_list_tmp.unshift({
        name: "no_user_domain",
        label: this.$t("settings.internal_authentication"),
        value: "-",
      });
      this.ldap_domain_list = ldap_domain_list_tmp;
      // set already_set to true if the configuration is not empty
      if (
        this.wikiName &&
        this.username &&
        this.password &&
        this.userFullName &&
        this.email
      ) {
        this.already_set = true;
      }
      this.loading.getConfiguration = false;

      this.focusElement("host");
    },
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;

      if (!this.wikiName) {
        this.error.wiki_name = "common.required";

        if (isValidationOk) {
          this.focusElement("wikiName");
        }
        isValidationOk = false;
      }

      if (!this.username) {
        this.error.username = "common.required";

        if (isValidationOk) {
          this.focusElement("username");
        }
        isValidationOk = false;
      }

      if (!this.password) {
        this.error.password = "common.required";

        if (isValidationOk) {
          this.focusElement("password");
        }
        isValidationOk = false;
      }

      if (!this.email) {
        this.error.email = "common.required";

        if (isValidationOk) {
          this.focusElement("email");
        }
        isValidationOk = false;
      }

      if (this.email && !/^\S+@\S+$/.test(this.email)) {
        this.error.email = "settings.email_format";

        if (isValidationOk) {
          this.focusElement("email");
        }
        isValidationOk = false;
      }

      if (!this.userFullName) {
        this.error.user_full_name = "common.required";

        if (isValidationOk) {
          this.focusElement("userFullName");
        }
        isValidationOk = false;
      }

      if (!this.host) {
        this.error.host = "common.required";

        if (isValidationOk) {
          this.focusElement("host");
        }
        isValidationOk = false;
      }

      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = "settings." + validationError.error;

        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$off(taskAction + "-validation-failed");
      this.core.$root.$once(
        taskAction + "-validation-failed",
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.configureModuleCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            wiki_name: this.wikiName,
            username: this.username,
            password: this.password,
            user_full_name: this.userFullName,
            email: this.email,
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            ldap_domain: this.ldap_domain == "-" ? "" : this.ldap_domain,
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.core.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}
</style>
