<script>
import CreateEditView from '@shell/mixins/create-edit-view';
import { LabeledInput } from '@components/Form/LabeledInput';
import { RadioGroup } from '@components/Form/Radio';

export default {
  name: 'HarvesterVMForceDeletePolicy',

  components: { LabeledInput, RadioGroup },

  mixins: [CreateEditView],

  data() {
    let parseDefaultValue = {};

    try {
      parseDefaultValue = JSON.parse(this.value.value);
    } catch (error) {
      parseDefaultValue = JSON.parse(this.value.default);
    }

    return { parseDefaultValue };
  },

  created() {
    this.update();
  },

  methods: {
    update() {
      const value = JSON.stringify(this.parseDefaultValue);

      this.value['value'] = value;
    }
  },

  watch: {
    value: {
      handler(neu) {
        const parseDefaultValue = JSON.parse(neu.value);

        this['parseDefaultValue'] = parseDefaultValue;
      },
      deep: true
    }
  }
};
</script>

<template>
  <div
    class="row"
    @input="update"
  >
    <div class="col span-12">
      <RadioGroup
        v-model:value="parseDefaultValue.enable"
        class="mb-20"
        name="model"
        :options="[true,false]"
        :labels="[t('generic.enabled'), t('generic.disabled')]"
        @update:value="update"
      />

      <LabeledInput
        v-if="parseDefaultValue.enable"
        v-model:value.number="parseDefaultValue.period"
        class="mb-20"
        :mode="mode"
        label-key="harvester.setting.vmForceDeletionPolicy.period"
      />
    </div>
  </div>
</template>

<style lang="scss" scoped>
  :deep() .radio-group {
    display: flex;
    .radio-container {
      margin-right: 30px;
    }
  }
</style>
