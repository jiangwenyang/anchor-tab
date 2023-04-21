<script>
import AnchorTabNav from './AnchorTabNav.vue'
import throttle from 'lodash/throttle'

export default {
  name: 'AnchorTabs',
  components: {
    AnchorTabNav
  },
  props: {
    value: {
      type: String
    },
    beforeLeave: {
      type: Function,
      default: null
    }
  },

  provide() {
    return {
      rootTabs: this
    }
  },

  data() {
    return {
      currentName: this.value,
      panes: [],
      isScrolling: false // 是否处于滚动状态，用于防止手动点击tab切换时防止触发滚动监听
    }
  },

  watch: {
    value(value) {
      this.setCurrentName(value)
    }
  },

  methods: {
    // 收集pane实例
    calcPaneInstances(isForceUpdate = false) {
      if (this.$slots.default) {
        const paneSlots = this.$slots.default.filter(
          vnode =>
            vnode.tag &&
            vnode.componentOptions &&
            vnode.componentOptions.Ctor.options.name === 'AnchorTabPane'
        )
        // update indeed
        const panes = paneSlots.map(
          ({ componentInstance }) => componentInstance
        )
        const panesChanged = !(
          panes.length === this.panes.length &&
          panes.every((pane, index) => pane === this.panes[index])
        )
        if (isForceUpdate || panesChanged) {
          this.panes = panes
        }
      } else if (this.panes.length !== 0) {
        this.panes = []
      }
    },

    // 处理Tab点击
    handleTabClick(tab, tabName, event) {
      if (tab.disabled) return

      this.isScrolling = true
      this.setCurrentName(tabName)
      this.scrollToPane(tabName)
      this.$emit('tab-click', tab, event)
      setTimeout(() => {
        this.isScrolling = false
      }, 500)
    },

    // 设置当前
    setCurrentName(value) {
      const changeCurrentName = () => {
        this.currentName = value
        this.$emit('input', value)
      }
      if (this.currentName !== value && this.beforeLeave) {
        const before = this.beforeLeave(value, this.currentName)
        if (before && before.then) {
          before.then(
            () => {
              changeCurrentName()
              this.$refs.nav && this.$refs.nav.removeFocus()
            },
            () => {
              // https://github.com/ElemeFE/element/pull/14816
              // ignore promise rejection in `before-leave` hook
            }
          )
        } else if (before !== false) {
          changeCurrentName()
        }
      } else {
        changeCurrentName()
      }
    },

    // 滚动到指定Pane
    scrollToPane(tabName) {
      const containerEl = this.$refs['panelsContainer']
      const paneEl = document.getElementById(`anchor-tab-pane-${tabName}`)
      if (!(containerEl && paneEl)) {
        return
      }

      if (paneEl.scrollIntoView) {
        // 如果支持scrollIntoView，则直接使用
        paneEl.scrollIntoView({
          behavior: 'smooth'
        })
      } else {
        const offsetTopTop = paneEl.offsetTop
        const containerOffsetTop = containerEl.offsetTop

        containerEl.scroll({
          top: offsetTopTop - containerOffsetTop,
          behavior: 'smooth'
        })
      }
    },

    // 获取最近的面板名称
    getClosestPanelName() {
      const containerEl = this.$refs['panelsContainer']
      const paneEls = document.getElementsByClassName('anchor-tab-pane')
      if (!(containerEl && paneEls)) {
        return
      }
      const { top: containerTop } = containerEl.getBoundingClientRect()

      let minTop = null
      let closestPaneEl = null
      ;[...paneEls].forEach(el => {
        const { top } = el.getBoundingClientRect()
        const offsetTop = top - containerTop

        if (offsetTop < 0) {
          return
        }
        if (minTop === null || offsetTop <= minTop) {
          minTop = offsetTop
          closestPaneEl = el
        }
      })

      if (!closestPaneEl) {
        return
      }

      const closestPanelName = closestPaneEl.getAttribute(
        'data-anchor-tab-pane-name'
      )

      return closestPanelName
    },

    setClosestPane() {
      const closestPanelName = this.getClosestPanelName()
      closestPanelName && this.setCurrentName(closestPanelName)
    },

    // 处理容器滚动
    handleContainerScroll() {
      if (this.isScrolling) {
        return
      }
      this.setClosestPane()
    },

    // 创建一个高度为容器高度-16的幽灵元素，保证所有pane都可以滚动到容器顶部
    createGhost() {
      const containerEl = this.$refs['panelsContainer']
      const height = containerEl.offsetHeight

      const ghostEl = document.createElement('div')
      ghostEl.className = 'anchor-tab-pane__ghost'
      ghostEl.style.height = `${height * (2 / 3)}px`

      containerEl.appendChild(ghostEl)
    }
  },

  // eslint-disable-next-line no-unused-vars
  render(h) {
    const { handleTabClick, handleContainerScroll, currentName, panes } = this

    const navData = {
      props: {
        onTabClick: handleTabClick,
        currentName,
        panes
      },
      ref: 'nav'
    }
    const anchor = (
      <div class='anchor-tabs__nav'>
        <AnchorTabNav {...navData}></AnchorTabNav>
      </div>
    )
    const panels = (
      <div
        class='anchor-tabs__content'
        ref='panelsContainer'
        onScroll={throttle(handleContainerScroll, 100)}
      >
        {this.$slots.default}
      </div>
    )

    return <div class='anchor-tabs'>{[anchor, panels]}</div>
  },

  created() {
    this.$on('tab-nav-update', this.calcPaneInstances.bind(null, true))
  },

  mounted() {
    this.calcPaneInstances()
    this.createGhost()
    if (!this.value) {
      this.setClosestPane()
    }
  },

  updated() {
    this.calcPaneInstances()
  }
}
</script>

<style lang="scss" scoped>
.anchor-tabs {
  display: flex;
  height: 100%;
}
.anchor-tabs__nav {
  min-width: 200px;
}
.anchor-tabs__content {
  flex: 1;
  overflow: auto;
}
</style>
