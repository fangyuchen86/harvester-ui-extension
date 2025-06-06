<script>
import Tabbed from '@shell/components/Tabbed';
import Tab from '@shell/components/Tabbed/Tab';
import CruResource from '@shell/components/CruResource';
import { LabeledInput } from '@components/Form/LabeledInput';
import NameNsDescription from '@shell/components/form/NameNsDescription';
import FileSelector, { createOnSelected } from '@shell/components/form/FileSelector';

import { randomStr } from '@shell/utils/string';
import CreateEditView from '@shell/mixins/create-edit-view';

export default {
  name: 'HarvesterEditKeypair',

  emits: ['update:value'],

  components: {
    Tab,
    Tabbed,
    CruResource,
    LabeledInput,
    FileSelector,
    NameNsDescription
  },

  mixins: [CreateEditView],

  inheritAttrs: false,

  props: {
    value: {
      type:     Object,
      required: true,
    }
  },

  data() {
    if ( !this.value.spec ) {
      this.value.spec = {};
      this.value.metadata = { name: '' };
    }

    return {
      publicKey:    this.value.spec.publicKey || '',
      randomString: '',
    };
  },

  watch: {
    publicKey(neu) {
      const trimNeu = neu.trim();

      this.value.spec.publicKey = trimNeu;

      const splitSSH = trimNeu.split(/\s+/);

      if (splitSSH.length === 3 && !this.value.metadata.name) {
        const keyComment = splitSSH[2];

        this.randomString = randomStr(10).toLowerCase();
        this.value.metadata.name = keyComment.includes('@') ? keyComment.split('@')[0] : keyComment;
      }
    }
  },

  methods: { onKeySelected: createOnSelected('publicKey') },
};
</script>

<template>
  <CruResource
    :done-route="doneRoute"
    :resource="value"
    :mode="mode"
    :errors="errors"
    :apply-hooks="applyHooks"
    @finish="save"
    @error="e=>errors=e"
  >
    <div class="header mb-20">
      <FileSelector
        v-if="isCreate"
        class="btn btn-sm bg-primary mt-10"
        :label="t('generic.readFromFile')"
        accept=".pub"
        @selected="onKeySelected"
      />
    </div>

    <NameNsDescription
      ref="nd"
      :key="randomString"
      :value="value"
      :mode="mode"
      @update:value="$emit('update:value', $event)"
    />

    <Tabbed
      v-bind="$attrs"
      class="mt-15"
      :side-tabs="true"
    >
      <Tab
        name="basic"
        :label="t('harvester.sshKey.tabs.basics')"
        :weight="1"
        class="bordered-table"
      >
        <LabeledInput
          v-model:value="publicKey"
          type="multiline"
          :mode="mode"
          :min-height="160"
          :label="t('harvester.sshKey.keypair')"
          required
        />
      </Tab>
    </Tabbed>
  </CruResource>
</template>

<style lang="scss" scoped>
.header {
  display: flex;
  justify-content: flex-end;
}
</style>
