<script>
export default {
  name: 'NovncConsoleItem',

  emits: ['send-keys', 'update'],

  props: {
    items: {
      type:     Object,
      required: true,
      default:  () => {
        return {};
      }
    },

    path: {
      type:     Array,
      required: true,
      default:  () => {
        return [];
      }
    },

    pos: {
      type:     Number,
      required: true,
      default:  0,
    }
  },

  methods: {
    keysDown(key, pos) {
      this.addKeys({ key, pos });
      this.$emit('send-keys');
    },

    addKeys({ key, pos }) {
      this.$emit('update', { key, pos });
    },

    sendKeys() {
      this.$emit('send-keys');
    },

    getOpenStatus(key, pos) {
      return this.path[pos] === key;
    }
  }
};
</script>

<template>
  <ul class="list-unstyled dropdown combination-keys__container">
    <li
      v-for="(item, key) in items"
      :key="key"
    >
      <v-dropdown
        v-if="!!item.keys"
        placement="right-start"
        trigger="click"
        :container="false"
      >
        <span
          :class="{ open: getOpenStatus(key, pos) }"
          class="p-10 hand"
          @click="addKeys({ key, pos })"
        >{{ item.label }}</span>

        <template #popper>
          <novnc-console-item
            :items="item.keys"
            :path="path"
            :pos="pos+1"
            @update="addKeys"
            @send-keys="sendKeys"
          />
        </template>
      </v-dropdown>

      <span
        v-else
        class="p-10 hand"
        @click="keysDown(key, pos)"
      >{{ item.label }}</span>
    </li>
  </ul>
</template>

<style lang="scss" scoped>
  .combination-keys__container {
    max-width: 60px;

    DIV, SPAN {
      display: block;
      text-align: center;
    }

    SPAN {
      border-radius: 3px;

      &:hover, &.open {
        color: var(--primary-hover-text);
        background: var(--primary-hover-bg);
      }
    }
  }
</style>
