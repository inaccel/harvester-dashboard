<script>
import UnitInput from '@shell/components/form/UnitInput';
import InputOrDisplay from '@shell/components/InputOrDisplay';

export default {
  name:       'HarvesterEditFpga',
  components: { UnitInput, InputOrDisplay },

  props: {
    bittwareIa420f: {
      type:    String,
      default: null
    },
    bittwareIa840f: {
      type:    String,
      default: null
    },
    intelPacA10: {
      type:    String,
      default: null
    },
    intelPacS10: {
      type:    String,
      default: null
    },
    mode: {
      type:    String,
      default: 'create',
    },
    disabled: {
      type:    Boolean,
      default: false
    }
  },

  data() {
    return {
      localBittwareIa420f: this.bittwareIa420f,
      localBittwareIa840f: this.bittwareIa840f,
      localIntelPacA10:    this.intelPacA10,
      localIntelPacS10:    this.intelPacS10
    };
  },

  computed: {
    bittwareIa420fDisplay() {
      if (this.localBittwareIa420f) {
        return `${ this.localBittwareIa420f } FPGAs`;
      } else {
        return '-';
      }
    },

    bittwareIa840fDisplay() {
      if (this.localBittwareIa840f) {
        return `${ this.localBittwareIa840f } FPGAs`;
      } else {
        return '-';
      }
    },

    intelPacA10Display() {
      if (this.localIntelPacA10) {
        return `${ this.localIntelPacA10 } FPGAs`;
      } else {
        return '-';
      }
    },

    intelPacS10Display() {
      if (this.localIntelPacS10) {
        return `${ this.localIntelPacS10 } FPGAs`;
      } else {
        return '-';
      }
    }
  },

  watch: {
    bittwareIa420f(neu) {
      this.localBittwareIa420f = neu;
    },
    bittwareIa840f(neu) {
      this.localBittwareIa840f = neu;
    },
    intelPacA10(neu) {
      this.localIntelPacA10 = neu;
    },
    intelPacS10(neu) {
      this.localIntelPacS10 = neu;
    }
  },

  methods: {
    change() {
      this.$emit('updateFpga', this.localBittwareIa420f, this.localBittwareIa840f, this.localIntelPacA10, this.localIntelPacS10);
    },

  }
};
</script>

<template>
  <div>
    <div class="row">
      <div class="col span-6">
        <InputOrDisplay :name="t('harvester.virtualMachine.input.bittwareIa420f')" :value="bittwareIa420fDisplay" :mode="mode" class="mb-10">
          <UnitInput
            v-model="localBittwareIa420f"
            v-int-number
            :label="t('harvester.virtualMachine.input.bittwareIa420f')"
            suffix="FPGAs"
            :delay="0"
            :output-modifier="true"
            :disabled="disabled"
            :mode="mode"
            class="mb-20"
            @input="change"
          />
        </InputOrDisplay>
      </div>

      <div class="col span-6">
        <InputOrDisplay :name="t('harvester.virtualMachine.input.bittwareIa840f')" :value="bittwareIa840fDisplay" :mode="mode" class="mb-10">
          <UnitInput
            v-model="localBittwareIa840f"
            v-int-number
            :label="t('harvester.virtualMachine.input.bittwareIa840f')"
            suffix="FPGAs"
            :delay="0"
            :output-modifier="true"
            :disabled="disabled"
            :mode="mode"
            class="mb-20"
            @input="change"
          />
        </InputOrDisplay>
      </div>
    </div>

    <div class="row">
      <div class="col span-6">
        <InputOrDisplay :name="t('harvester.virtualMachine.input.intelPacA10')" :value="intelPacA10Display" :mode="mode" class="mb-10">
          <UnitInput
            v-model="localIntelPacA10"
            v-int-number
            :label="t('harvester.virtualMachine.input.intelPacA10')"
            suffix="FPGAs"
            :delay="0"
            :output-modifier="true"
            :disabled="disabled"
            :mode="mode"
            class="mb-20"
            @input="change"
          />
        </InputOrDisplay>
      </div>

      <div class="col span-6">
        <InputOrDisplay :name="t('harvester.virtualMachine.input.intelPacS10')" :value="intelPacS10Display" :mode="mode" class="mb-10">
          <UnitInput
            v-model="localIntelPacS10"
            v-int-number
            :label="t('harvester.virtualMachine.input.intelPacS10')"
            suffix="FPGAs"
            :delay="0"
            :output-modifier="true"
            :disabled="disabled"
            :mode="mode"
            class="mb-20"
            @input="change"
          />
        </InputOrDisplay>
      </div>
    </div>
  </div>
</template>
