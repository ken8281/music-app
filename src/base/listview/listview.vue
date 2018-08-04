<template>
  <scroll 
  :data='data' 
  ref='listview' 
  id='listview'
  :listenScroll='listenScroll'
  :probeType='probeType'
  @scroll='scroll'
  >
    <ul>
      <li ref='listGroup' v-for='group in data' class="list-group">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li @click.stop='selectItem(item, $event)' v-for='item in group.items' class="list-group-item">
              <img class="avatar" v-lazy="item.avatar">
              <span class='name'>{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" 
    @touchstart.stop.prevent='onShortcutTouchStart' 
    @touchmove.stop.prevent='onShortcutTouchMove'
    @touchend.stop='onShortcutTouchEnd'
    >
      <ul>
        <li 
        v-for='(item, index) in shortcutList' 
        class="item" 
        :data-index='index' 
        :data-group-title='fixedTitle'
        :class='{current: currentIndex === index}' 
        >{{item}}</li>
      </ul>
    </div>
    <div v-show='fixedTitle' class="list-fixed" ref='fixed'>
      <h2 class="fixed-title">{{fixedTitle}}</h2>
    </div>
    <div v-show='!data.length' class="loading-container">
      <loading/>
    </div>
    <div v-show='groupTitle' class="group-title">{{groupTitle}}</div>
  </scroll>
</template>
<script type="text/ecmascript-6">
import Scroll from 'base/scroll/scroll'
import {getData} from 'common/js/dom'
import Loading from 'base/loading/loading'

const ANCHOR_HEIGHT = 18
const TITLE_HEIGHT = 30

export default {
  created () {
    this.touch = {}
    this.listenScroll = true
    this.probeType = 3
    this.listHeight = []
  },
  data () {
    return {
      scrollY: -1,
      currentIndex: 0,
      diff: -1,
      groupTitle: ''
    }
  },
  props: {
    data: {
      type: Array,
      default: []
    }
  },
  computed: {
    shortcutList () {
      return this.data.map((group) => {
        return group.title.substring(0, 1)
      })
    },
    fixedTitle () {
      if(this.scrollY > 0) {
        return ''
      }
      let group = this.data[this.currentIndex]
      return group ? group.title : ''
    }
  },
  methods: {
    selectItem(item, e) {
      if (!e._constructed) {
        return
      }
      this.$emit('select', item)
    },
    onShortcutTouchStart (e) {
      let anchorIndex = parseInt(getData(e.target, 'index'))
      let firstTouch = e.touches[0]
      this.touch.y1 = firstTouch.pageY
      this.touch.anchorIndex = anchorIndex
      this._scrollTo(anchorIndex)
    },
    onShortcutTouchMove (e) {
      let firstTouch = e.touches[0]
      this.touch.y2 = firstTouch.pageY
      let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0
      let anchorIndex = this.touch.anchorIndex + delta
      this._scrollTo(anchorIndex)
      this.groupTitle = this.fixedTitle.substring(0, 1)
    },
    onShortcutTouchEnd (e) {
      this.groupTitle = ''
    },
    scroll (pos) {
      this.scrollY = pos.y
    },
    refresh () {
      this.$refs.listview.refresh()
    },
    _scrollTo (index) {
      //边界点击的处理
      if (!index && index !== 0) {
        return
      }
      //touchmove越界的处理
      if (index < 0) {
        index = 0
      } else if (index > this.listHeight.length -2) {
        index = this.listHeight.length -2
        //listHeight比导航栏的length多1，这里要获取高度的上限
      }
      //导航栏高亮处理
      this.scrollY = -this.listHeight[index]

      this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)
    },
    _caculateHeight () {
      this.listHeight = []
      let height = 0
      this.listHeight.push(height)
      let list = this.$refs.listGroup
      for(let i = 0; i < list.length; i++) {
        height += list[i].clientHeight
        this.listHeight.push(height)
      }
    }
  },
  components: {
    Scroll,
    Loading
  },
  watch: {
    data () {
      this.$nextTick(() => {
        this._caculateHeight()
      })
    },
    scrollY (newY) {
      let listHeight = this.listHeight
      //顶部下拉
      if (newY > 0) {
          this.currentIndex = 0
          return
        }

      for (let i = 0; i < listHeight.length - 1; i++) {
        let height1 = listHeight[i]
        let height2 = listHeight[i+1]
        //中间部分
        if (-newY >= height1 && -newY < height2) {
          this.currentIndex = i
          this.diff = height2 + newY
          return
        }
      //底部上拉
      this.currentIndex = listHeight.length - 2
      }
    },
    diff (val) {
      let fixedTop = (val > 0 && val < TITLE_HEIGHT) ? val - TITLE_HEIGHT : 0
      if (this.fixedTop === fixedTop) {
        return
      }
      this.fixedTop = fixedTop
      this.$refs.fixed.style.transform = `translate3d(0, ${fixedTop}px, 0)`
    }
  }
}
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/variable"

#listview
  position: relative
  width: 100%
  height: 100%
  overflow: hidden
  background-color: $color-background
  .list-group
    padding-bottom: 30px
    .list-group-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background-color: $color-highlight-background
    .list-group-item
      display: flex
      align-items: center
      padding: 20px 0 0 30px
      .avatar
        width: 50px
        height: 50px
        border-radius: 50%
      .name
        margin-left: 20px
        color: $color-text-l
        font-size: $font-size-medium
  .list-shortcut
    position: absolute
    z-index: 30
    right: 0
    top: 50%
    transform: translateY(-50%)
    width: 20px
    padding: 20px 0
    border-radius: 10px
    text-align: center
    background-color: $color-background-d
    font-family: Helvetica
    .item
      padding: 3px;
      line-height: 1
      color: $color-text-l
      font-size: $font-size-small
      &.current
        color: $color-theme
  .list-fixed
    position: absolute
    top: 0
    left: 0
    width: 100%
    .fixed-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background-color: $color-highlight-background
  .loading-container
    position: absolute
    width: 100%
    top: 50%
    transform: translateY(-50%)
  .group-title
    position: absolute
    width: 40px
    height: 40px
    top: 50%
    left: 50%
    transform: translate(-50%, -50%)
    background-color: $color-background-d
    border-radius: 50%
    line-height: 40px
    text-align: center
</style>
