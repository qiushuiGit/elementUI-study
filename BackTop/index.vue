<template>
  <transition :name="transitionName">
    <div
      v-if="visible"
      :style="{
        'right': styleRight,
        'bottom': styleBottom
      }"
      class="el-backtop"
      @click.stop="handleClick"
    >
      <!-- 插槽，设置默认样式 -->
      <slot>
        <el-icon name="caret-top" />
      </slot>
    </div>
  </transition>
</template>

<script>
// 防抖节流插件：https://www.npmjs.com/package/throttle-debounce
// 大家可以通过 npm 这里我是直接引入 js
import throttle from './js/throttle'

const cubic = value => Math.pow(value, 3)
const easeInOutCubic = value => value < 0.5
  ? cubic(value * 2) / 2
  : 1 - cubic((1 - value) * 2) / 2

export default {
  name: 'ElBacktop',
  props: {
    // 滚动高度
    visibilityHeight: {
      type: Number,
      default: 200
    },
    // 触发滚动的对象
    target: [String],
    // 页面右边距距离
    right: {
      type: Number,
      default: 40
    },
    // 页面底部距离
    bottom: {
      type: Number,
      default: 40
    },
    // 过渡动画
    transitionName: {
      type: String,
      default: 'el-fade-in'
    }
  },

  data() {
    return {
      el: null, // 触发滚动的对象
      container: null,
      visible: false // 控制组件显示
    }
  },
  computed: {
    // 利用计算属性，监听 bottom 和 right 的值
    styleBottom() {
      return `${this.bottom}px`
    },
    styleRight() {
      return `${this.right}px`
    }
  },
  mounted() {
    this.init() // 初始化
    this.throttledScrollHandler = throttle(300, this.onScroll) // 生成节流函数并返回
    this.container.addEventListener('scroll', this.throttledScrollHandler) // 监听 scroll 事件
  },
  // 销毁钩子
  beforeDestroy() {
    // 移除 scroll 事件
    this.container.removeEventListener('scroll', this.throttledScrollHandler)
  },

  methods: {
    // 初始化，主要是获取滚动的对象
    init() {
      this.container = document
      this.el = document.documentElement
      if (this.target) {
        this.el = document.querySelector(this.target)
        if (!this.el) {
          throw new Error(`target is not existed: ${this.target}`)
        }
        this.container = this.el
      }
    },
    // 监听滚动值，用于显示和隐藏回到顶部按钮组件
    onScroll() {
      const scrollTop = this.el.scrollTop
      this.visible = scrollTop >= this.visibilityHeight
    },
    // 点击事件
    handleClick(e) {
      this.scrollToTop()
      this.$emit('click', e) // 触发父级监听事件函数 click
    },
    // 滚动函数
    scrollToTop() {
      const el = this.el // 滚动对象
      const beginTime = Date.now() // 开始滚动的时间
      const beginValue = el.scrollTop // 滚动距离
      // 执行动画
      const rAF = window.requestAnimationFrame || (func => setTimeout(func, 16))
      // 更新动画的回调函数
      const frameFunc = () => {
        // 控制动画进度
        const progress = (Date.now() - beginTime) / 500
        // progress 大于 1，则滚动动画执行完成，滚动条回到顶部
        if (progress < 1) {
          el.scrollTop = beginValue * (1 - easeInOutCubic(progress))
          rAF(frameFunc)
        } else {
          el.scrollTop = 0
        }
      }
      // 回调，持续执行滚动动画
      rAF(frameFunc)
    }
  }
}
</script>
