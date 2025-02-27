
<script>
import { LabeledInput } from '@components/Form/LabeledInput';
import { _CREATE } from '@shell/config/query-params';
import RadioGroup from '@components/Form/Radio/RadioGroup.vue';

export const DATA_DIR_RADIO_OPTIONS = {
  DEFAULT: 'defaultDataDir',
  COMMON:  'commonBaseDataDir',
  CUSTOM:  'customDataDir',
};

export const DEFAULT_COMMON_BASE_PATH = '/var/lib/rancher';

export const DEFAULT_SUBDIRS = {
  AGENT:           'agent',
  PROVISIONING:    'provisioning',
  K8S_DISTRO_RKE2: 'rke2',
  K8S_DISTRO_K3S:  'k3s',
};

export default {
  name:       'DirectoryConfig',
  components: {
    LabeledInput,
    RadioGroup
  },
  props: {
    mode: {
      type:     String,
      required: true,
    },

    k8sVersion: {
      type:     String,
      required: true,
    },

    value: {
      type:     Object,
      required: true,
    },
  },
  data() {
    let dataConfigRadioValue = DATA_DIR_RADIO_OPTIONS.DEFAULT;
    let k8sDistroSubDir = DEFAULT_SUBDIRS.K8S_DISTRO_RKE2;

    if (this.k8sVersion && this.k8sVersion.includes('k3s')) {
      k8sDistroSubDir = DEFAULT_SUBDIRS.K8S_DISTRO_K3S;
    }

    if (this.mode !== _CREATE) {
      if (this.value?.systemAgent?.length || this.value?.provisioning?.length || this.value?.k8sDistro?.length) {
        dataConfigRadioValue = DATA_DIR_RADIO_OPTIONS.CUSTOM;
      }
    }

    return {
      DATA_DIR_RADIO_OPTIONS,
      dataConfigRadioValue,
      k8sDistroSubDir,
      commonConfig: '',
    };
  },
  watch: {
    commonConfig(neu) {
      if (neu && neu.length && this.dataConfigRadioValue === DATA_DIR_RADIO_OPTIONS.COMMON) {
        this.value.systemAgent = `${ neu }/${ DEFAULT_SUBDIRS.AGENT }`;
        this.value.provisioning = `${ neu }/${ DEFAULT_SUBDIRS.PROVISIONING }`;
        this.value.k8sDistro = `${ neu }/${ this.k8sDistroSubDir }`;
      }
    },
    k8sVersion: {
      handler(neu) {
        if (neu && neu.includes('k3s')) {
          this.k8sDistroSubDir = DEFAULT_SUBDIRS.K8S_DISTRO_K3S;
        } else {
          this.k8sDistroSubDir = DEFAULT_SUBDIRS.K8S_DISTRO_RKE2;
        }

        if (this.value.k8sDistro) {
          this.value.k8sDistro = `${ neu }/${ this.k8sDistroSubDir }`;
        }
      }
    }
  },
  computed: {
    dataConfigRadioOptions() {
      const defaultDataDirOption = {
        value: DATA_DIR_RADIO_OPTIONS.DEFAULT,
        label: this.t('cluster.directoryConfig.radioInput.defaultLabel')
      };
      const customDataDirOption = {
        value: DATA_DIR_RADIO_OPTIONS.CUSTOM,
        label: this.t('cluster.directoryConfig.radioInput.customLabel')
      };

      if (this.mode === _CREATE) {
        return [
          defaultDataDirOption,
          { value: DATA_DIR_RADIO_OPTIONS.COMMON, label: this.t('cluster.directoryConfig.radioInput.commonLabel') },
          customDataDirOption
        ];
      } else {
        return [
          defaultDataDirOption,
          customDataDirOption
        ];
      }
    }
  },
  methods: {
    handleRadioInput(val) {
      switch (val) {
      case DATA_DIR_RADIO_OPTIONS.DEFAULT:
        if (this.mode === _CREATE) {
          this.commonConfig = '';
        }
        this.value.systemAgent = '';
        this.value.provisioning = '';
        this.value.k8sDistro = '';

        this.dataConfigRadioValue = DATA_DIR_RADIO_OPTIONS.DEFAULT;
        break;
      case DATA_DIR_RADIO_OPTIONS.COMMON:
        this.commonConfig = DEFAULT_COMMON_BASE_PATH;

        this.dataConfigRadioValue = DATA_DIR_RADIO_OPTIONS.COMMON;
        break;
      // default is custom config
      default:
        if (this.mode === _CREATE) {
          this.commonConfig = '';
        }

        this.value.systemAgent = '';
        this.value.provisioning = '';
        this.value.k8sDistro = '';

        this.dataConfigRadioValue = DATA_DIR_RADIO_OPTIONS.CUSTOM;
        break;
      }
    }
  },
};
</script>

<template>
  <div class="row">
    <div class="col span-8">
      <h3 class="mb-20">
        {{ t('cluster.directoryConfig.title') }}
      </h3>
      <RadioGroup
        :value="dataConfigRadioValue"
        class="mb-10"
        :mode="mode"
        :options="dataConfigRadioOptions"
        name="directory-config-radio"
        data-testid="rke2-directory-config-radio-input"
        @update:value="handleRadioInput"
      />
      <LabeledInput
        v-if="dataConfigRadioValue === DATA_DIR_RADIO_OPTIONS.COMMON"
        v-model:value="commonConfig"
        class="mb-20"
        :mode="mode"
        :label="t('cluster.directoryConfig.common.label')"
        :tooltip="t('cluster.directoryConfig.common.tooltip')"
        data-testid="rke2-directory-config-common-data-dir"
      />
      <div v-if="dataConfigRadioValue === DATA_DIR_RADIO_OPTIONS.CUSTOM">
        <LabeledInput
          v-model:value="value.systemAgent"
          class="mb-20"
          :mode="mode"
          :label="t('cluster.directoryConfig.systemAgent.label')"
          :tooltip="t('cluster.directoryConfig.systemAgent.tooltip')"
          data-testid="rke2-directory-config-systemAgent-data-dir"
        />
        <LabeledInput
          v-model:value="value.provisioning"
          class="mb-20"
          :mode="mode"
          :label="t('cluster.directoryConfig.provisioning.label')"
          :tooltip="t('cluster.directoryConfig.provisioning.tooltip')"
          data-testid="rke2-directory-config-provisioning-data-dir"
        />
        <LabeledInput
          v-model:value="value.k8sDistro"
          class="mb-20"
          :mode="mode"
          :label="t('cluster.directoryConfig.k8sDistro.label')"
          :tooltip="t('cluster.directoryConfig.k8sDistro.tooltip')"
          data-testid="rke2-directory-config-k8sDistro-data-dir"
        />
      </div>
      <div class="mb-40" />
    </div>
  </div>
</template>

<style lang="scss" scoped>

</style>
