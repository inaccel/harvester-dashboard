<script>
import ConsumptionGauge from '@shell/components/ConsumptionGauge';
import { NODE, POD } from '@shell/config/types';

export default {
  name:       'HarvesterFpgaUsed',
  components: { ConsumptionGauge },

  props:      {
    value: {
      type:     String,
      default: ''
    },

    row: {
      type:     Object,
      required: true
    },

    resourceName: {
      type:     String,
      default: ''
    },

    intelPacA10Only: {
      type:    Boolean,
      default: false,
    },

    intelPacS10Only: {
      type:    Boolean,
      default: false,
    },

    showReserved: {
      type:    Boolean,
      default: false,
    },
  },

  data() {
    return {};
  },

  computed: {
    node() {
      const inStore = this.$store.getters['currentProduct'].inStore;
      const node = this.$store.getters[`${ inStore }/byId`](NODE, this.row.id);

      return node;
    },

    pods() {
      const inStore = this.$store.getters['currentProduct'].inStore;
      const allPods = this.$store.getters[`${ inStore }/all`](POD);

      return allPods.filter(pod => pod.spec.nodeName === this.node.name);
    },

    intelPacA10Usage() {
      return this.pods.filter(pod => pod.status.phase !== 'Succeeded' && pod.status.phase !== 'Failed').map((pod) => {
        let limit = 0;

        pod.spec.containers?.forEach((container) => {
          const quantity = Number.parseInt(container.resources?.limits?.['intel/pac_a10'] || '0');

          limit += quantity;
        });
        pod.spec.initContainers?.forEach((container) => {
          const quantity = Number.parseInt(container.resources?.limits?.['intel/pac_a10'] || '0');

          if (quantity > limit) {
            limit = quantity;
          }
        });

        return limit;
      }).reduce((nodeUsage, limit) => nodeUsage + limit, 0.0);
    },

    intelPacA10Allocatable() {
      return Number.parseInt(this.node.status.allocatable['intel/pac_a10'] || '0');
    },

    intelPacA10Capacity() {
      return Number.parseInt(this.node.status.capacity['intel/pac_a10'] || '0');
    },

    intelPacS10Usage() {
      return this.pods.filter(pod => pod.status.phase !== 'Succeeded' && pod.status.phase !== 'Failed').map((pod) => {
        let limit = 0;

        pod.spec.containers?.forEach((container) => {
          const quantity = Number.parseInt(container.resources?.limits?.['intel/pac_s10'] || '0');

          limit += quantity;
        });
        pod.spec.initContainers?.forEach((container) => {
          const quantity = Number.parseInt(container.resources?.limits?.['intel/pac_s10'] || '0');

          if (quantity > limit) {
            limit = quantity;
          }
        });

        return limit;
      }).reduce((nodeUsage, limit) => nodeUsage + limit, 0.0);
    },

    intelPacS10Allocatable() {
      return Number.parseInt(this.node.status.capacity['intel/pac_s10'] || '0');
    },

    intelPacS10Capacity() {
      return Number.parseInt(this.node.status.capacity['intel/pac_s10'] || '0');
    },

    fpgaUsage() {
      if (this.intelPacA10Only) {
        return this.intelPacA10Usage;
      }
      if (this.intelPacS10Only) {
        return this.intelPacS10Usage;
      }

      return this.intelPacA10Usage + this.intelPacS10Usage;
    },

    fpgaAllocatable() {
      if (this.intelPacA10Only) {
        return this.intelPacA10Allocatable;
      }
      if (this.intelPacS10Only) {
        return this.intelPacS10Allocatable;
      }

      return this.intelPacA10Allocatable + this.intelPacS10Allocatable;
    },

    fpgaCapacity() {
      if (this.intelPacA10Only) {
        return this.intelPacA10Capacity;
      }
      if (this.intelPacS10Only) {
        return this.intelPacS10Capacity;
      }

      return this.intelPacA10Capacity + this.intelPacS10Capacity;
    },
  },
};
</script>

<template>
  <div>
    <div
      v-if="showReserved"
    >
      <ConsumptionGauge
        :capacity="fpgaCapacity"
        :used="fpgaAllocatable"
        :resource-name="resourceName"
      >
        <template #title="{amountTemplateValues, formattedPercentage}">
          <span>
            {{ t('clusterIndexPage.hardwareResourceGauge.reserved') }}
          </span>
          <span>
            {{ t('node.detail.glance.consumptionGauge.amount', amountTemplateValues) }}
            <span class="ml-10 percentage">/&nbsp;{{ formattedPercentage }}
            </span>
          </span>
        </template>
      </ConsumptionGauge>
    </div>
    <ConsumptionGauge
      :capacity="fpgaCapacity"
      :used="fpgaUsage"
      :class="{
        'mt-10': showReserved,
      }"
    >
      <template #title="{amountTemplateValues, formattedPercentage}">
        <span>
          {{ t('clusterIndexPage.hardwareResourceGauge.used') }}
        </span>
        <span>
          {{ t('node.detail.glance.consumptionGauge.amount', amountTemplateValues) }}
          <span class="ml-10 percentage">/&nbsp;{{ formattedPercentage }}
          </span>
        </span>
      </template>
    </ConsumptionGauge>
  </div>
</template>
