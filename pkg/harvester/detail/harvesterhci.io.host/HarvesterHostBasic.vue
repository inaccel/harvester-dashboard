<script>
import LabelValue from '@shell/components/LabelValue';
import { formatSi, exponentNeeded, UNITS } from '@shell/utils/units';
import { HCI as HCI_ANNOTATIONS } from '@pkg/harvester/config/labels-annotations';
import { LONGHORN, METRIC } from '@shell/config/types';
import { Banner } from '@components/Banner';
import HarvesterCPUUsed from '../../formatters/HarvesterCPUUsed';
import HarvesterFPGAUsed from '../../formatters/HarvesterFPGAUsed';
import HarvesterMemoryUsed from '../../formatters/HarvesterMemoryUsed';
import HarvesterStorageUsed from '../../formatters/HarvesterStorageUsed';

const COMPLETE = 'complete';
const PROMOTE_RESTART = 'promoteRestart';
const PROMOTE_SUCCEED = 'promoteSucceed';

export default {
  name: 'BasicNode',

  components: {
    Banner,
    LabelValue,
    HarvesterCPUUsed,
    HarvesterFPGAUsed,
    HarvesterMemoryUsed,
    HarvesterStorageUsed,
  },

  props: {
    value: {
      type:     Object,
      required: true,
    },

    metrics: {
      type:     Object,
      required: false,
      default:  () => {
        return null;
      }
    },

    mode: {
      type:     String,
      required: false,
      default:  'view'
    },
  },

  computed: {
    customName() {
      return this.value.metadata?.annotations?.[HCI_ANNOTATIONS.HOST_CUSTOM_NAME];
    },

    consoleUrl() {
      const consoleUrl = this.value.metadata?.annotations?.[HCI_ANNOTATIONS.HOST_CONSOLE_URL];
      let value = consoleUrl;

      if (!consoleUrl) {
        return '';
      }

      if (!consoleUrl.startsWith('http://') && !consoleUrl.startsWith('https://')) {
        value = `http://${ consoleUrl }`;
      }

      return {
        display: consoleUrl,
        value
      };
    },

    cpuTotal() {
      let out = 0;

      if (this.metrics) {
        out = this.metrics.cpuCapacity;
      }

      return out;
    },

    cpuUsage() {
      let out = 0;

      if (this.metrics) {
        out = this.metrics.cpuUsage;
      }

      return out;
    },

    memoryTotal() {
      let out = 0;

      if (this.metrics) {
        out = this.metrics.memoryCapacity;
      }

      return out;
    },

    memoryUsage() {
      let out = 0;

      if (this.metrics) {
        out = this.metrics.memoryUsage;
      }

      return out;
    },

    cpuUnits() {
      return 'C';
    },

    memoryUnits() {
      const exponent = exponentNeeded(this.memoryTotal, 1024);

      return `${ UNITS[exponent] }iB`;
    },

    bittwareIa420fCapacity() {
      return Number.parseInt(this.value.status.capacity['bittware/ia420f'] || '0');
    },

    bittwareIa840fCapacity() {
      return Number.parseInt(this.value.status.capacity['bittware/ia840f'] || '0');
    },

    intelPacA10Capacity() {
      return Number.parseInt(this.value.status.capacity['intel/pac_a10'] || '0');
    },

    intelPacS10Capacity() {
      return Number.parseInt(this.value.status.capacity['intel/pac_s10'] || '0');
    },

    fpgaCapacity() {
      return this.bittwareIa420fCapacity + this.bittwareIa840fCapacity + this.intelPacA10Capacity + this.intelPacS10Capacity;
    },

    nodeType() {
      if (this.value.isEtcd) {
        return this.t('harvester.host.detail.etcd');
      }

      if (this.value.isMaster) {
        return this.t('harvester.host.detail.management');
      }

      return this.t('harvester.host.detail.compute');
    },

    lastUpdateTime() {
      return this.value.status?.conditions?.[0]?.lastHeartbeatTime;
    },

    nodeRoleState() {
      if (!this.value.isEtcd) {
        const promoteStatus = this.value.metadata?.annotations?.[HCI_ANNOTATIONS.PROMOTE_STATUS];

        if (promoteStatus === COMPLETE) {
          const isExistRoleStatus = this.value.metadata?.labels?.[HCI_ANNOTATIONS.NODE_ROLE_MASTER] !== undefined || this.value.metadata?.labels?.[HCI_ANNOTATIONS.NODE_ROLE_CONTROL_PLANE] !== undefined;

          return this.t(`harvester.host.promote.${ isExistRoleStatus ? PROMOTE_SUCCEED : PROMOTE_RESTART }`);
        }
      }

      return null;
    },

    hasMetricNodeSchema() {
      const inStore = this.$store.getters['currentProduct'].inStore;

      return !!this.$store.getters[`${ inStore }/schemaFor`](METRIC.NODE);
    },

    hasLonghornSchema() {
      const inStore = this.$store.getters['currentProduct'].inStore;

      return !!this.$store.getters[`${ inStore }/schemaFor`](LONGHORN.NODES);
    },
  },

  methods: {
    memoryFormatter(value) {
      const exponent = exponentNeeded(this.memoryTotal, 1024);

      const formatOptions = {
        addSuffix:   false,
        increment:   1024,
        minExponent: exponent
      };

      return formatSi(value, formatOptions);
    },
  }
};
</script>

<template>
  <div class="host-detail">
    <Banner
      v-if="value.isKVMDisable"
      color="error"
      label-key="harvester.host.detail.kvm.disableMessage"
    />
    <h3>{{ t('harvester.host.tabs.overview') }}</h3>
    <div class="row mb-20">
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.customName')" :value="customName" />
      </div>
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.hostIP')" :value="value.internalIp" />
      </div>
    </div>

    <div class="row mb-20">
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.os')" :value="value.status.nodeInfo.osImage" />
      </div>
      <div class="col span-6">
        <div class="role">
          <LabelValue :name="t('harvester.host.detail.role')">
            <template #value>
              {{ nodeType }}
              <span
                v-if="nodeRoleState"
                class="text-warning ml-20"
              >
                {{ nodeRoleState }}
              </span>
            </template>
          </LabelValue>
        </div>
      </div>
    </div>

    <div class="row mb-20">
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.create')" :value="value.metadata.creationTimestamp" />
      </div>
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.update')" :value="lastUpdateTime" />
      </div>
    </div>

    <div class="row mb-20">
      <div class="col span-6">
        <LabelValue :name="t('harvester.host.detail.consoleUrl')" :value="consoleUrl.value">
          <a slot="value" :href="consoleUrl.value" target="_blank">{{ consoleUrl.display }}</a>
        </LabelValue>
      </div>
    </div>

    <div v-if="hasMetricNodeSchema">
      <hr class="divider" />
      <h3>{{ t('harvester.host.tabs.monitor') }}</h3>
      <div class="row mb-20">
        <div
          class="col"
          :class="{
            'span-4': hasLonghornSchema,
            'span-6': !hasLonghornSchema,
          }"
        >
          <HarvesterCPUUsed
            :row="value"
            :resource-name="t('node.detail.glance.consumptionGauge.cpu')"
            :show-used="true"
          />
        </div>
        <div
          class="col"
          :class="{
            'span-4': hasLonghornSchema,
            'span-6': !hasLonghornSchema,
          }"
        >
          <HarvesterMemoryUsed
            :row="value"
            :resource-name="t('node.detail.glance.consumptionGauge.memory')"
            :show-used="true"
          />
        </div>
        <div
          v-if="hasLonghornSchema"
          class="col span-4"
        >
          <HarvesterStorageUsed
            :row="value"
            :resource-name="t('harvester.host.detail.storage')"
            :show-reserved="true"
          />
        </div>
      </div>
      <div v-if="fpgaCapacity" class="row mb-20">
        <div
          v-if="bittwareIa420fCapacity"
          class="col span-4"
        >
          <HarvesterFPGAUsed
            :row="value"
            :resource-name="t('harvester.dashboard.hardwareResourceGauge.bittwareIa420f')"
            :bittware-ia420f-only="true"
            :show-reserved="true"
          />
        </div>
        <div
          v-if="bittwareIa840fCapacity"
          class="col span-4"
        >
          <HarvesterFPGAUsed
            :row="value"
            :resource-name="t('harvester.dashboard.hardwareResourceGauge.bittwareIa840f')"
            :bittware-ia840f-only="true"
            :show-reserved="true"
          />
        </div>
        <div
          v-if="intelPacA10Capacity"
          class="col span-4"
        >
          <HarvesterFPGAUsed
            :row="value"
            :resource-name="t('harvester.dashboard.hardwareResourceGauge.intelPacA10')"
            :intel-pac-a10-only="true"
            :show-reserved="true"
          />
        </div>
        <div
          v-if="intelPacS10Capacity"
          class="col span-4"
        >
          <HarvesterFPGAUsed
            :row="value"
            :resource-name="t('harvester.dashboard.hardwareResourceGauge.intelPacS10')"
            :intel-pac-s10-only="true"
            :show-reserved="true"
          />
        </div>
      </div>
    </div>

    <hr class="section-divider" />
    <h3>{{ t('harvester.host.detail.more') }}</h3>
    <div class="row mb-20">
      <div class="col span-4">
        <LabelValue :name="t('harvester.host.detail.uuid')" :value="value.status.nodeInfo.systemUUID" />
      </div>

      <div class="col span-4">
        <LabelValue :name="t('harvester.host.detail.kernel')" :value="value.status.nodeInfo.kernelVersion" />
      </div>

      <div class="col span-4">
        <LabelValue :name="t('harvester.host.detail.containerRuntime')" :value="value.status.nodeInfo.containerRuntimeVersion" />
      </div>
    </div>
    <div
      v-if="value.manufacturer || value.serialNumber || value.model"
      class="row mb-20"
    >
      <div
        v-if="value.manufacturer"
        class="col span-4"
      >
        <LabelValue
          :name="t('harvester.host.detail.manufacturer')"
          :value="value.manufacturer"
        />
      </div>
      <div
        v-if="value.serialNumber"
        class="col span-4"
      >
        <LabelValue
          :name="t('harvester.host.detail.serialNumber')"
          :value="value.serialNumber"
        />
      </div>
      <div
        v-if="value.model"
        class="col span-4"
      >
        <LabelValue
          :name="t('harvester.host.detail.model')"
          :value="value.model"
        />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.role {
  display: flex;
}
</style>
