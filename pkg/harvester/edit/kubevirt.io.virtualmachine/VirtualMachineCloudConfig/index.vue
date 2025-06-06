<script>
import { mapGetters } from 'vuex';

import { LabeledInput } from '@components/Form/LabeledInput';
import YamlEditor, { EDITOR_MODES } from '@shell/components/YamlEditor';
import ModalWithCard from '@shell/components/ModalWithCard';

import { CONFIG_MAP } from '@shell/config/types';
import { HCI as HCI_ANNOTATIONS } from '@pkg/harvester/config/labels-annotations';
import { _VIEW } from '@shell/config/query-params';
import DataTemplate from './DataTemplate';

const _NEW = '_NEW';

export default {
  emits: ['updateUserData', 'updateNetworkData', 'updateDataTemplateId'],

  components: {
    DataTemplate, YamlEditor, LabeledInput, ModalWithCard
  },

  props: {
    mode: {
      type:    String,
      default: 'create'
    },
    namespace: {
      type:    String,
      default: ''
    },
    osType: {
      type:    String,
      default: ''
    },
    userScript: {
      type:    String,
      default: ''
    },
    networkScript: {
      type:    String,
      default: ''
    },
    viewCode: {
      type:    Boolean,
      default: false
    }
  },

  data() {
    return {
      errors:            [],
      templateType:      '',
      cloudTemplate:     '',
      cloudTemplateName: '',
      configUserId:      '',
      configNetworkId:   '',
      optionUser:        [],
      optionNetwork:     [],
      isOpen:            false
    };
  },

  async fetch() {
    const configs = await this.$store.dispatch('harvester/findAll', { type: CONFIG_MAP });

    const optionUser = [];
    const optionNetwork = [];

    for (const config of configs) {
      if (config.metadata?.labels?.[HCI_ANNOTATIONS.CLOUD_INIT] === 'user') {
        optionUser.push({
          label: config?.id,
          value: config?.id
        });
      }

      if (config.metadata?.labels?.[HCI_ANNOTATIONS.CLOUD_INIT] === 'network') {
        optionNetwork.push({
          label: config?.id,
          value: config?.id
        });
      }
    }

    optionUser.unshift({
      label: this.t('harvester.virtualMachine.cloudConfig.createNew'),
      value: _NEW,
    });

    optionNetwork.unshift({
      label: this.t('harvester.virtualMachine.cloudConfig.createNew'),
      value: _NEW,
    });

    this.optionUser = optionUser;
    this.optionNetwork = optionNetwork;
  },

  computed: {
    ...mapGetters({ t: 'i18n/t' }),

    createTypeLable() {
      const label = `harvester.virtualMachine.cloudConfig.${ this.templateType }.label`;

      return this.t(label);
    },

    editorMode() {
      return this.isView ? EDITOR_MODES.VIEW_CODE : EDITOR_MODES.EDIT_CODE;
    },

    isView() {
      return this.mode === _VIEW;
    },
  },

  methods: {
    updateValue() {
      this.$refs['userTemplate'].updateValue();
      this.$refs['networkTemplate'].updateValue();
    },

    update(value, type) {
      if (type === 'user') {
        this.$emit('updateUserData', value);
      }
      if (type === 'network') {
        this.$emit('updateNetworkData', value);
      }
    },

    updateTemplateId(type, id) {
      this.$emit('updateDataTemplateId', type, id);
    },

    show(templateType) {
      this.templateType = templateType;
      this.isOpen = true;
    },

    async save(buttonCb) {
      this.errors = [];

      if (!this.cloudTemplateName) {
        this.errors.push(this.t('validation.required', { key: this.t('harvester.virtualMachine.input.name') }, true));
        buttonCb(false);

        return;
      }

      if (!this.cloudTemplate) {
        const keyLabel = this.templateType === 'user' ? 'harvester.cloudTemplate.userData' : 'harvester.cloudTemplate.networkData';

        this.errors.push(this.t('validation.required', { key: this.t(keyLabel) }, true));
        buttonCb(false);

        return;
      }

      const templateValue = await this.$store.dispatch('harvester/create', {
        type:     CONFIG_MAP,
        metadata: {
          labels:    { [HCI_ANNOTATIONS.CLOUD_INIT]: this.templateType },
          name:      this.cloudTemplateName,
          namespace: this.namespace
        },
        data: { cloudInit: this.cloudTemplate },
      });

      try {
        const res = await templateValue.save();

        if (res.id) {
          this.templateType === 'user' ? this.configUserId = res.id : this.configNetworkId = res.id;
          this.$fetch();
        }
        buttonCb(true);
        this.cancel();
      } catch (err) {
        this.errors = [err.message];
        buttonCb(false);
      }
    },

    cancel() {
      this.cloudTemplate = '';
      this.cloudTemplateName = '';
      this['errors'] = [];
      this.isOpen = false;
    },

    refresh() {
      this.$refs['userTemplate'].refresh();
      this.$refs['networkTemplate'].refresh();
    },
  }
};
</script>

<template>
  <div>
    <h2 v-if="!isView">
      {{ t('harvester.virtualMachine.cloudConfig.title') }}
    </h2>

    <div class="mb-20">
      <DataTemplate
        ref="userTemplate"
        :value="userScript"
        :os-type="osType"
        type="user"
        :view-code="viewCode"
        :mode="mode"
        :config-id="configUserId"
        :options="optionUser"
        @show="show"
        @update="update"
        @updateTemplateId="updateTemplateId"
      />
    </div>

    <div class="mb-20">
      <DataTemplate
        ref="networkTemplate"
        :value="networkScript"
        type="network"
        :os-type="osType"
        :view-code="viewCode"
        :mode="mode"
        :config-id="configNetworkId"
        :options="optionNetwork"
        @show="show"
        @update="update"
      />
    </div>

    <ModalWithCard
      v-if="isOpen"
      name="createCloudTemplate"
      width="40%"
      :errors="errors"
      @finish="save"
      @close="cancel"
    >
      <template #title>
        {{ t('harvester.virtualMachine.cloudConfig.createTemplateTitle', { name: createTypeLable }) }}
      </template>

      <template #content>
        <LabeledInput
          v-model:value="cloudTemplateName"
          :label="t('harvester.virtualMachine.input.name')"
          class="mb-20"
          required
          @keydown.native.enter.prevent="()=>{}"
        />

        <div class="yaml">
          <div class="resource-yaml">
            <YamlEditor
              ref="createTemplate"
              v-model:value="cloudTemplate"
              class="yaml-editor"
              :editor-mode="editorMode"
            />
          </div>
        </div>
      </template>
    </ModalWithCard>
  </div>
</template>

<style lang="scss" scoped>
$yaml-height: 350px;

.yaml {
  height: $yaml-height;
  overflow: auto;
}

:deep() .resource-yaml {
  flex: 1;
  display: flex;
  flex-direction: column;

  & .yaml-editor{
    flex: 1;
    min-height: $yaml-height;

    & .code-mirror .CodeMirror {
      min-height: $yaml-height;
    }
  }
}
</style>
