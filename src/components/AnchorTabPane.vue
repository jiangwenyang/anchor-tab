<template>
  <div
    class="anchor-tab-pane"
    :data-anchor-tab-pane-name="name"
    :id="`anchor-tab-pane-${name}`"
  >
    <slot></slot>
  </div>
</template>
<script>
export default {
  name: 'AnchorTabPane',
  componentName: 'AnchorTabPane',
  props: {
    label: String,
    name: String
  },
  updated() {
    this.$parent.$emit('tab-nav-update')
  }
}
</script>
