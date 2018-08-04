<template>
	<div id="player" v-show='playlist.length'>
    <transition name='normal'
                @enter='enter'
                @after-enter='afterEnter'
                @leave='leave'
                @after-leave='afterLeave'
    >
      <div class="normal-player" v-show='fullScreen'>
        <div class="background">
          <img width='100%' height='100%' :src='currentSong.image'>
        </div>
        <div class="top">
          <div @click='back' class="back">
            <i class="icon-back"></i>
          </div>
          <h1 class="title">{{currentSong.name}}</h1>
          <h2 class="subtitle">{{currentSong.singer}}</h2>
        </div>
        <div class="middle"
             @touchstart.prevent="middleTouchStart"
             @touchmove.prevent="middleTouchMove"
             @touchend="middleTouchEnd"
        >
          <div class="middle-l" ref='middleL'>
            <div ref='cdWrapper' class="cd-wrapper">
              <div ref='cd' class="cd">
                <img :class='cdCls' ref='cdImg' class="image" :src='currentSong.image'>
              </div>
            </div>
            <div class="playing-lyric-wrapper">
              <div class="playing-lyric">{{playingLyric}}</div>
            </div>
          </div>
          <scroll class="middle-r" ref="lyricList" :data="currentLyric && currentLyric.lines">
            <div class="lyric-wrapper">
              <div v-if="currentLyric">
                <p ref="lyricLine"
                   class="text"
                   :class="{'current': currentLineNum ===index}"
                   v-for="(line,index) in currentLyric.lines" v-html='line.txt'></p>
              </div>
              <div class="pure-music" v-show='isPureMusic'>
                <p>{{pureMusicLyric}}</p>
              </div>
            </div>
          </scroll>
        </div>
        <div class="bottom">
          <div class="dot-wrapper">
            <span :class='{active: currentShow==="cd"}' class="dot"></span>
            <span :class='{active: currentShow==="lyric"}' class="dot"></span>
          </div>
          <div class="progress-wrapper">
            <span class="time time-l">{{currentTime | format}}</span>
            <div class="progress-bar-wrapper">
              <progress-bar ref='progressBar' @percentchange='onProgressBarChange' :percent='percent'/>
            </div>
            <span class="time time-r">{{currentSong.duration | format}}</span>
          </div>
          <div class="operators">
            <div class="icon i-left">
              <i @click='changeMode' :class="iconMode"></i>
            </div>
            <div class="icon i-left" :class='disableCls'>
              <i @click='prev' class="icon-prev"></i>
            </div>
            <div class="icon i-center" :class='disableCls'>
              <i @click='togglePlaying' :class="playIcon"></i>
            </div>
            <div class="icon i-right" :class='disableCls'>
              <i @click='next' class="icon-next"></i>
            </div>
            <div class="icon i-right">
              <i @click.stop='toggleFavorite(currentSong)' :class='getFavoriteIcon(currentSong)' class="icon"></i>
            </div>
          </div>
        </div>
      </div>
    </transition>
    <transition name='mini'>
      <div @click='open' class="mini-player" v-show='!fullScreen'>
        <div class="icon">
          <div class="imgWrapper" ref='miniCd'>
            <img ref='miniCdImg' :class='cdCls' width='40' height='40' :src='currentSong.image'>
          </div>
        </div>
        <div class="text">
          <h2 class="name">{{currentSong.name}}</h2>
          <p class="desc">{{currentSong.singer}}</p>
        </div>
        <div class="control">
          <progress-circle :percent='percent' :radius='radius'>
            <i @click.stop='togglePlaying' :class="miniIcon" class='icon-mini'></i>
          </progress-circle>
        </div>
        <div @click.stop='showPlaylist' class="control">
          <i class="icon-playlist"></i>
        </div>
      </div>
    </transition>
    <playlist ref='playlist'/>
    <audio @ended='end' @timeupdate='updateTime' @play='ready' @error='error' ref='audio' :src="currentSong.url"></audio>
  </div>
</template>

<script type='text/ecmascript-6'>
import {mapGetters, mapMutations,mapActions} from 'vuex'
import animations from 'create-keyframe-animation'
import {prefixStyle} from 'common/js/dom'
import ProgressBar from 'base/progress-bar/progress-bar'
import ProgressCircle from 'base/progress-circle/progress-circle'
import {playMode} from 'common/js/config'
import {shuffle} from 'common/js/util'
import Lyric from 'lyric-parser'
import Scroll from 'base/scroll/scroll'
import Playlist from 'components/playlist/playlist'
import {playerMixin} from 'common/js/mixin'

const transform = prefixStyle('transform')
const transitionDuration = prefixStyle('transitionDuration')
const timeExp = /\[(\d{2}):(\d{2}):(\d{2})\]/g

export default {
  mixins: [playerMixin],
  data () {
    return {
      songReady: false,
      currentTime: 0,
      radius: 32,
      currentLyric: null,
      currentLineNum: 0,
      currentShow: 'cd',
      playingLyric: '',
      isPureMusic: false,
      pureMusicLyric: ''
    }
  },
  created () {
    this.touch = {}
  },
  methods: {
    back () {
      this.setFullScreen(false)
    },
    open () {
      this.setFullScreen(true)
      //mini暂停后，进入normal调整进度条的值
      this.$nextTick(() => {
        this.$refs.lyricList.refresh()
        this.$refs.progressBar.setProgressOffset(this.percent)
      })
    },
    togglePlaying () {
      if (!this.songReady) {
        return
      }
      if(this.currentLyric) {
        this.currentLyric.togglePlay()
      }
      this.setPlayingState(!this.playing)
    },
    prev () {
      if (!this.songReady) {
        return
      }
      if (this.playlist.length === 1) {
        this.loop()
        return
      } else {
          let index = this.currentIndex - 1
          if (index < 0) {
            index = this.playlist.length - 1
          }
          this.setCurrentIndex(index)
          if (!this.playing) {
            this.togglePlaying()
          }
          this.songReady = false
        }
    },
    next () {
      if (!this.songReady) {
        return
      }
      if (this.playlist.length === 1) {
        this.loop()
        return
      } else {
          let index = this.currentIndex + 1
          if (index >= this.playlist.length) {
            index = 0
          }
          this.setCurrentIndex(index)
          if (!this.playing) {
            this.togglePlaying()
          }
          this.songReady = false
        }
    },
    ready () {
      this.songReady = true
      this.canLyricPlay = true
      this.savePlayHistory(this.currentSong)
      //同步歌词
      if (this.currentLyric && !this.isPureMusic) {
        this.currentLyric.seek(this.currentTime * 1000)
      }
    },
    error () {
      this.songReady = true
    },
    updateTime (e) {
      this.currentTime = e.target.currentTime
    },
    end () {
      this.currentTime = 0
      if (this.mode === playMode.loop) {
        this.loop()
      } else {
        this.next()
      }
    },
    loop () {
      let audio = this.$refs.audio
      audio.currentTime = 0
      audio.play()
      if(this.currentLyric) {
        this.currentLyric.seek(0)
      }
    },
    onProgressBarChange (percent) {
      let currentTime = this.currentSong.duration * percent
      this.$refs.audio.currentTime = currentTime
      if (!this.playing) {
        this.togglePlaying()
      }
      if(this.currentLyric) {
        this.currentLyric.seek(currentTime * 1000)
      }
    },
    // changeMode () {
    //   let mode = (this.mode + 1) % 3
    //   this.setPlayMode(mode)
    //   let list = null
    //   if (this.mode ===playMode.random) {
    //     list = shuffle(this.sequenceList)
    //   } else {
    //     list = this.sequenceList
    //   }
    //   this.resetCurrentIndex(list)
    //   this.setPlayList(list)
    // },
    // resetCurrentIndex (list) {
    //   let index = list.findIndex((item) => {
    //     return item.id === this.currentSong.id
    //   })
    //   this.setCurrentIndex(index)
    // },
    getLyric () {
      this.currentSong.getLyric().then((lyric) => {
        //快速切换时，异步操作的lyric比song.lyric赋值要慢，使new Lyric()执行了两次，歌词有一瞬间会乱掉
        if (this.currentSong.lyric !== lyric) {
          return
        }
        this.currentLyric = new Lyric(lyric, this.handleLyric)
        this.isPureMusic = !this.currentLyric.lines.length
        if(this.isPureMusic) {
          this.pureMusicLyric = this.currentLyric.lrc.replace(timeExp, '').trim()
          this.playingLyric = this.pureMusicLyric
        } else {
          //同步歌词
          if(this.playing && this.canLyricPlay) {
            this.currentLyric.seek(this.currentTime * 1000)
          }
        }
      }).catch(() => {
        this.currentLyric = null
        this.playingLyric = ''
        this.currentLineNum = 0
      })
    },
    handleLyric({lineNum, txt}) {
      this.currentLineNum = lineNum
      if (lineNum > 5) {
        let lineEl = this.$refs.lyricLine[lineNum - 5]
        this.$refs.lyricList.scrollToElement(lineEl, 1000)
      } else {
        this.$refs.lyricList.scrollTo(0, 0, 1000)
      }
      this.playingLyric = txt
    },
    middleTouchStart(e) {
        //touch.initiated用于判定touch事件的触发，但个人认为无需设置这个值
        this.touch.initiated = true
        // 用来判断是否是一次移动
        if (this.touch.moved) {
          this.touch.moved = false
        }
        const touch = e.touches[0]
        this.touch.startX = touch.pageX
        this.touch.startY = touch.pageY
      },
      middleTouchMove(e) {
        if (!this.touch.initiated) {
          return
        }
        const touch = e.touches[0]
        const deltaX = touch.pageX - this.touch.startX
        const deltaY = touch.pageY - this.touch.startY
        if (Math.abs(deltaY) > Math.abs(deltaX)) {
          return
        }
        if (!this.touch.moved) {
          this.touch.moved = true
        }
        const left = this.currentShow === 'cd' ? 0 : -window.innerWidth
        const offsetWidth = Math.min(0, Math.max(-window.innerWidth, left + deltaX))
        this.touch.percent = Math.abs(offsetWidth / window.innerWidth)
        this.$refs.lyricList.$el.style[transform] = `translate3d(${offsetWidth}px,0,0)`
        this.$refs.lyricList.$el.style[transitionDuration] = 0
        this.$refs.middleL.style.opacity = 1 - this.touch.percent
        this.$refs.middleL.style[transitionDuration] = 0
      },
      middleTouchEnd() {
        if (!this.touch.moved) {
          return
        }
        let offsetWidth
        let opacity
        if (this.currentShow === 'cd') {
          if (this.touch.percent > 0.1) {
            offsetWidth = -window.innerWidth
            opacity = 0
            this.currentShow = 'lyric'
          } else {
            offsetWidth = 0
            opacity = 1
          }
        } else {
          if (this.touch.percent < 0.9) {
            offsetWidth = 0
            this.currentShow = 'cd'
            opacity = 1
          } else {
            offsetWidth = -window.innerWidth
            opacity = 0
          }
        }
        const time = 300
        this.$refs.lyricList.$el.style[transform] = `translate3d(${offsetWidth}px,0,0)`
        this.$refs.lyricList.$el.style[transitionDuration] = `${time}ms`
        this.$refs.middleL.style.opacity = opacity
        this.$refs.middleL.style[transitionDuration] = `${time}ms`
        this.touch.initiated = false
      },
      showPlaylist () {
        this.$refs.playlist.show()
      },
      //切换播放器时，歌曲cd图的inner变回默认值
      syncWrapperTransform (wrapper, inner) {
        if (!this.$refs[wrapper]) {
          return
        }
        let imageWrapper = this.$refs[wrapper]
        let image = this.$refs[inner]
        let wTransform = window.getComputedStyle(imageWrapper)[transform]
        let iTransform = window.getComputedStyle(image)[transform]
        if (wTransform === 'none') {
          imageWrapper.style[transform] = iTransform
        } else {
          imageWrapper.style[transform] = iTransform.concat(' ', wTransform)
        }

      },
    // _pad (num, n = 2) {
    //   let len =  num.toString().length
    //   return len < n ? `0${num}` : num
    // },
    enter (el, done) {
      const {x, y, scale} = this.getPosAndScale

      let animation = {
        0: {
          transform: `translate3d(${x}px, ${y}px, 0) scale(${scale})`
        },
        60: {
          transform: `translate3d(0, 0, 0) scale(1.1)`
        },
        100: {
          transform: `translate3d(0, 0, 0) scale(1)`
        }
      }

      animations.registerAnimation({
        name: 'move',
        animation,
        presets: {
          duration: 400,
          easing: 'linear'
        }
      })

      animations.runAnimation(this.$refs.cdWrapper, 'move', done)
    },
    afterEnter (el) {
      animations.unregisterAnimation('move')
      this.$refs.cdWrapper.style.animation = ''
    },
    leave (el, done) {
      const {x, y, scale} = this.getPosAndScale
      let cdWrapper = this.$refs.cdWrapper
      cdWrapper.style.transition = 'all 0.4s'
      cdWrapper.style[transform] = `translate3d(${x}px, ${y}px, 0) scale(${scale})`
      cdWrapper.addEventListener('transitionend', done)
    },
    afterLeave (el) {
      let cdWrapper = this.$refs.cdWrapper
      cdWrapper.style.transition = ''
      cdWrapper.style[transform] = ''
    },
    ...mapMutations({
      setFullScreen: 'SET_FULL_SCREEN',
      // setPlayingState: 'SET_PLAYING_STATE',
      // setCurrentIndex: 'SET_CURRENT_INDEX',
      // setPlayMode: 'SET_PLAY_MODE',
      // setPlayList: 'SET_PLAYLIST'
    }),
    ...mapActions([
        'savePlayHistory'
      ])
  },
  computed: {
    getPosAndScale () {
      const targetWidth = 40
      const paddingLeft = 40
      const paddingBottom = 30
      const paddingTop = 80
      const width = window.innerWidth * 0.8
      const scale = targetWidth / width
      const x = -(window.innerWidth / 2 -paddingLeft)
      const y = window.innerHeight - paddingTop - width / 2 - paddingBottom
      return {
        x,
        y,
        scale
      }
    },
    playIcon () {
      return this.playing ? 'icon-pause' : 'icon-play'
    },
    miniIcon () {
      return this.playing ? 'icon-pause-mini' : 'icon-play-mini'
    },
    cdCls () {
      return this.playing ? 'play' : 'pause'
    },
    disableCls () {
      return this.songReady ? '' : 'disable'
    },
    percent () {
      return this.currentTime / this.currentSong.duration
    },
    // iconMode () {
    //   let icon = ''
    //   switch(this.mode) {
    //      case playMode.sequence:
    //        icon = 'icon-sequence'
    //        break;
    //      case playMode.loop: 
    //        icon = 'icon-loop'
    //        break;
    //      case playMode.random: 
    //        icon = 'icon-random'
    //        break;
    //    }
    //    return icon
    // },
    ...mapGetters([
        'fullScreen',
        'playing',
        'currentIndex',
        // 'playlist',
        // 'currentSong',
        // 'mode',
        // 'sequenceList'
      ])
  },
  watch: {
    currentSong (newSong, oldSong) {
      if (!newSong.id || newSong.id === oldSong.id) {
        return
      }
      // this.songReady = false
      this.canLyricPlay = false
      if (this.currentLyric) {
        this.currentLyric.stop()
        this.currentLyric = null
        this.currentTime = 0
        this.playingLyric = ''
        this.currentLineNum = 0
      }
      clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.$refs.audio.play();
        this.getLyric()
      }, 1000)
    },
    playing (val) {
      let audio = this.$refs.audio
      this.$nextTick(() => {
        val ? audio.play() : audio.pause()
      })
      if (!val) {
        if(this.fullScreen) {
          this.syncWrapperTransform('cd', 'cdImg')
        } else {
          this.syncWrapperTransform('miniCd', 'miniCdImg')
        }
      }
    }
  },
  // fullScreen(newVal) {
  // 无法触发fullscreen的watch,原因未知的bug,下面的功能由open函数触发
  //   console.log(this.fullScreen);
  //   if (newVal) {
  //     this.$nextTick(() => {console.log(newVal);
  //       this.$refs.lyricList.refresh()
  //       this.$refs.progressBar.setProgressOffset(this.percent)
  //     })
  //   }
  // },
  filters: {
    format (time) {
      time = time | 0
      let minute = time / 60 | 0
      let second = time % 60
      if (second < 10) {
        second = `0${second}`
      }
      return `${minute}:${second}`
    }
  },
  components: {
    ProgressBar,
    ProgressCircle,
    Scroll,
    Playlist
  }
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import "~common/stylus/variable"
@import "~common/stylus/mixin"

#player
  .normal-player
    position: fixed
    left: 0
    right: 0
    top: 0
    bottom: 0
    z-index: 150
    background-color: $color-background
    .background
      position: absolute
      left: 0
      top: 0
      width: 100%
      height: 100%
      z-index: -1
      opacity: 0.6
      filter: blur(20px)
    .top
      position: relative
      margin-bottom: 25px
      .back
        position: absolute
        top: 0
        left: 6px
        z-index: 50
        .icon-back
          display: block
          padding: 9px
          font-size: $font-size-large-x
          color: $color-theme
          transform: rotate(-90deg)
      .title
        width: 70%
        margin: 0 auto
        line-height: 40px
        text-align: center
        no-wrap()
        font-size: $font-size-large
        color: $color-text
      .subtitle
        line-height: 20px
        text-align: center
        font-size: $font-size-medium
        color: $color-text
    .middle
      position: fixed
      width: 100%
      top: 80px
      bottom: 170px
      white-space: nowrap
      font-size: 0
      .middle-l
        display: inline-block
        vertical-align: top
        position: relative
        width: 100%
        height: 0
        padding-top: 80%
        .cd-wrapper
          position: absolute
          left: 10%
          top: 0
          width: 80%
          height: 100%
          .cd
            width: 100%
            height: 100%
            border-radius: 50%
            .image
              position: absolute
              left: 0
              top: 0
              width: 100%
              height: 100%
              box-sizing: border-box
              border: 10px solid rgba(255, 255, 255, 0.1)
              border-radius: 50%
              &.play
                animation: rotate 20s linear infinite
              &.pause
                animation-play-state: paused
        .playing-lyric-wrapper
          width: 80%
          margin: 30px auto 0 auto
          overflow: hidden
          text-align: center
          .playing-lyric
            height: 20px
            line-height: 20px
            font-size: $font-size-medium
            color: $color-text-l
      .middle-r
          display: inline-block
          vertical-align: top
          width: 100%
          height: 100%
          overflow: hidden
          .lyric-wrapper
            width: 80%
            margin: 0 auto
            overflow: hidden
            text-align: center
            .text
              line-height: 32px
              color: $color-text-l
              font-size: $font-size-medium
              &.current
                color: $color-text
            .pure-music
              padding-top: 50%
              line-height: 32px
              color: $color-text-l
              font-size: $font-size-medium
    .bottom
      position: absolute
      bottom: 50px
      width: 100%
      .dot-wrapper
        text-align: center
        font-size: 0
        .dot
          display: inline-block
          vertical-align: middle
          margin: 0 4px
          width: 8px
          height: 8px
          border-radius: 50%
          background-color: $color-text-l
          &.active
            width: 20px
            border-radius: 5px
            background-color: $color-text-ll
      .progress-wrapper
        display: flex
        align-items: center
        width: 80%
        margin: 0 auto
        padding: 10px 0
        .time
          color: $color-text
          font-size: $font-size-small
          flex: 0 0 30px
          width: 30px
          line-height: 30px
          &.time-l
            text-align: left
          &.time-r
            text-align: right
        .progress-bar-wrapper
          flex: 1
      .operators
        display: flex
        align-items: center
        .icon
          flex: 1
          color: $color-theme
          &.disable
            color: $color-theme-d
          i
            font-size: 30px
        .i-left
          text-align: right
        .i-center
          padding: 0 20px
          text-align: center
          i
            font-size: 40px
        .i-right
          text-align: left
        .icon-favorite
          color: $color-sub-theme
    &.normal-enter-active, &.normal-leave-active
      transition: all 0.4s
      .top, .bottom
        transition: all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
    &.normal-enter, &.normal-leave-to
      opacity: 0
      .top
        transform: translate3d(0, -100px, 0)
      .bottom
        transform: translate3d(0, 100px, 0)
  .mini-player
    display: flex
    align-items: center
    position: fixed
    left: 0
    bottom: 0
    z-index: 180
    width: 100%
    height: 60px
    background-color: $color-highlight-background
    &.mini-enter-active, &.mini-leave-active
      transition: opacity 0.4s
    &.mini-enter, &.mini-leave-to
      opacity: 0
    .icon
      flex: 0 0 40px
      height: 40px
      width: 40px
      padding: 0 10px 0 20px
      .imgWrapper
          height: 100%
          width: 100%
        img
          border-radius: 50%
          &.play
            animation: rotate 10s linear infinite
          &.pause
            animation-play-state: paused
    .text
      display: flex
      flex-direction: column
      justify-content: center
      flex: 1
      line-height: 20px
      overflow: hidden
      .name
        margin-bottom: 2px
        no-wrap()
        font-size: $font-size-medium
        color: $color-text
      .desc
        no-wrap()
        font-size: $font-size-small
        color: $color-text-d
    .control
      flex: 0 0 30px
      width: 30px
      padding: 0 10px
      .icon-play-mini, .icon-pause-mini, .icon-playlist
        font-size: 30px
        color: $color-theme-d
      .icon-mini
        font-size: 32px
        position: absolute
        left: 0
        top: 0
@keyframes rotate
  0%
    transform: rotate(0)
  100%
    transform: rotate(360deg)
</style>