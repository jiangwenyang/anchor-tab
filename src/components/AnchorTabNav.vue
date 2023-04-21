<template>
  <ul class="anchor-tab-nav-list">
    <li
      v-for="pane in panes"
      :key="pane.name"
      :class="[
        'anchor-tab-nav-item',
        {
          'is-active': pane.name === currentName
        }
      ]"
      @click="event => onTabClick(pane, pane.name, event)"
    >
      <div
        class="anchor-tab-nav-item__content"
        :title="pane.label || pane.name"
      >
        {{ pane.label || pane.name }}
      </div>
    </li>
  </ul>
</template>

<script>
export default {
  name: 'AnchorTabNav',
  props: {
    panes: {
      type: Array,
      default: () => []
    },
    currentName: {
      type: String,
      default: ''
    },

    onTabClick: {
      type: Function,
      default: () => {}
    }
  }
}
</script>

<style lang="scss" scoped>
$item-height: 40px;
$line-height: 60px;
.anchor-tab-nav-list {
  height: 100%;
  list-style: none;
  padding: 20px;
  overflow: auto;
}
.anchor-tab-nav-item {
  position: relative;
  border: 1px solid transparent;
  border-radius: 4px;
  height: $item-height;
  line-height: 40px;
  transition: all 0.3s ease-out;
  cursor: pointer;
  .anchor-tab-nav-item__content {
    padding-right: 10px;
    max-width: 200px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  &:hover,
  &.is-active {
    color: #3daeff;
    border-color: #3daeff;
    background: rgba(61, 174, 255, 0.2);
  }
  &.is-active .anchor-tab-nav-item__content::after {
    background: #3daeff;
  }
  &::before,
  &::after {
    content: '';
    height: $line-height;
    width: 1px;
    background: #aac5e1;
    position: absolute;
    left: 22px;
  }
  &::before {
    top: 0;
    transform: translate3d(0, -100%, 0);
    margin-top: $item-height/2;
  }
  &::after {
    top: 50%;
  }
}
.anchor-tab-nav-item + .anchor-tab-nav-item {
  margin-top: $line-height * 0.6;
}
.anchor-tab-nav-item__content {
  position: relative;
  padding-left: 40px;
  &::before,
  &::after {
    content: '';
    position: absolute;
    left: 16px;
    top: 50%;
    transform: translate3d(0, -50%, 0);
    border-radius: 50%;
  }
  &::before {
    height: 10px;
    width: 10px;
    background: #fff;
    z-index: 1;
    margin-left: 2px;
  }
  &::after {
    height: 14px;
    width: 14px;
    background: #aac5e1;
  }
}
</style>
