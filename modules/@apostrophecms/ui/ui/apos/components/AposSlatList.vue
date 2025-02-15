
<template>
  <div ref="root">
    <draggable
      class="apos-slat-list"
      tag="ol"
      role="list"
      :list="next"
      :move="onMove"
      v-bind="dragOptions"
      @start="isDragging=true"
      @end="isDragging=false"
      :id="listId"
    >
      <transition-group type="transition" name="apos-flip-list">
        <AposSlat
          v-for="item in next"
          class="apos-slat-list__item"
          @remove="remove"
          @engage="engage"
          @disengage="disengage"
          @select="select"
          @move="move"
          @item-clicked="$emit('item-clicked', item)"
          :key="item._id"
          :item="item"
          :selected="selected === item._id"
          :class="{'apos-slat-list__item--disabled' : disabled}"
          :disabled="disabled"
          :engaged="engaged === item._id"
          :parent="listId"
          :slat-count="next.length"
          :removable="removable"
          :has-relationship-schema="hasRelationshipSchema"
          :editor-label="editorLabel"
          :editor-icon="editorIcon"
        />
      </transition-group>
    </draggable>
  </div>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  name: 'AposSlatList',
  components: {
    draggable
  },
  props: {
    value: {
      type: Array,
      required: true
    },
    disabled: {
      type: Boolean,
      default: false
    },
    removable: {
      type: Boolean,
      default: true
    },
    selected: {
      type: String,
      default: null
    },
    hasRelationshipSchema: {
      type: Boolean,
      default: false
    },
    editorLabel: {
      type: String,
      default: null
    },
    editorIcon: {
      type: String,
      default: null
    }
  },
  emits: [ 'update', 'item-clicked', 'select', 'input' ],
  data() {
    return {
      isDragging: false,
      delayedDragging: false,
      engaged: null,
      next: this.value.slice()
    };
  },
  computed: {
    listId() {
      return `sortableList-${(Math.floor(Math.random() * Math.floor(10000)))}`;
    },
    dragOptions() {
      return {
        animation: 0,
        disabled: this.disabled || this.next.length <= 1,
        ghostClass: 'apos-is-dragging'
      };
    }
  },
  watch: {
    isDragging(newValue) {
      if (newValue) {
        this.delayedDragging = true;
        return;
      }
      this.$nextTick(() => {
        this.delayedDragging = false;
      });
    },
    value() {
      this.next = this.value.slice();
    },
    next(newValue, oldValue) {
      let equal = true;
      if (newValue.length === this.value.length) {
        for (let i = 0; (i < newValue.length); i++) {
          if ((newValue[i]._id !== this.value[i]._id) || (newValue[i].title !== this.value[i].title)) {
            equal = false;
            break;
          }
        }
      } else {
        equal = false;
      }
      if (!equal) {
        this.$emit('input', this.next);
      }
    }
  },
  methods: {
    engage(id) {
      this.engaged = id;
    },
    disengage(id) {
      this.engaged = null;
    },
    select(id) {
      this.$emit('select', id);
    },
    remove(item, focusNext) {
      const itemIndex = this.getIndex(item._id);
      this.next = this.next.filter(i => item._id !== i._id);
      if (focusNext && this.next[itemIndex]) {
        this.focusElement(this.next[itemIndex]._id);
      } else if (focusNext && this.next[itemIndex - 1]) {
        this.focusElement(this.next[itemIndex - 1]._id);
      }
      this.$emit('update', this.next);
    },
    move(id, dir) {
      const index = this.getIndex(id);
      const target = dir > 0 ? index + 1 : index - 1;
      if (this.next[target]) {
        this.next.splice(target, 0, this.next.splice(index, 1)[0]);
        this.focusElement(id);
        return this.$emit('update', this.next);
      }
    },
    getIndex(id) {
      let i = null;
      this.next.forEach((item, index) => {
        if (item._id === id) {
          i = index;
        }
      });
      return i;
    },
    focusElement(id) {
      if (this.$refs.root.querySelector(`[data-id="${id}"]`)) {
        this.$nextTick(() => {
          this.$refs.root.querySelector(`[data-id="${id}"]`).focus();
        });
      }
    },
    onMove({ relatedContext, draggedContext }) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return (
        (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      );
    }
  }
};
</script>

<style lang="scss" scoped>
  .apos-slat-list ::v-deep .apos-slat {
    margin-bottom: 5px;
    transition: all 0.4s;
    max-width: $input-max-width * 0.65;
  }

  .apos-flip-list-leave-to {
    opacity: 0;
  }

  .apos-is-dragging {
    opacity: 0.5;
    background: var(--a-base-4);
  }

  .apos-slat-list {
    @include apos-list-reset();
    min-height: 20px;
    max-width: $input-max-width;
  }

  .apos-slat-status {
    text-align: center;
  }

  .apos-slat-limit {
    @include type-help;
    margin: 10px 0 0;
    text-align: center;

    span {
      margin-right: 10px;
    }
  }
</style>
