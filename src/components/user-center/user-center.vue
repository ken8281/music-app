<template>
	<transition name='slider'>
		<div id="user-center">
			<div @click='back' class="back">
				<i class="icon-back"></i>
			</div>
			<div class="switches-wrapper">
				<switches @switch='switchItem' :currentIndex='currentIndex' :switches='switches' />
			</div>
			<div @click='random' ref='playBtn' class="play-btn">
				<i class="icon-play"></i>
				<span class="text">随机播放全部</span>
			</div>
			<div ref='listWrapper' class="list-wrapper">
				<scroll ref='favoriteList' :data='favoriteList' v-show='!currentIndex' class='list-scroll'>
					<div class='list-inner'>
						<song-list @select='selectSong' :songs='favoriteList' />
					</div>
				</scroll>
				<scroll ref='playList' :data='playHistory' v-show='currentIndex' class='list-scroll'>
					<div class='list-inner'>
						<song-list @select='selectSong' :songs='playHistory' />
					</div>
				</scroll>
			</div>
			<div class="no-result-wrapper">
				<no-result v-if='noResult' :title='noResultDesc' />
			</div>
		</div>
	</transition>
</template>

<script type="text/ecmascript-6">
import Switches from 'base/switches/switches'
import {mapGetters, mapActions} from 'vuex'
import Scroll from 'base/scroll/scroll'
import SongList from 'base/song-list/song-list'
import Song from 'common/js/song'
import {playlistMixin} from 'common/js/mixin'
import NoResult from 'base/no-result/no-result'

export default {
	mixins: [playlistMixin],
	data () {
		return {
			currentIndex: 0,
			switches: [
				{name: '我的收藏'},
				{name: '最近播放'}
			]
		}
	},
	methods: {
		switchItem (index) {
			this.currentIndex = index
		},
		selectSong (song) {
			this.insertSong(new Song(song))
		},
		back () {
			this.$router.back()
		},
		random () {
			let list = this.currentIndex ? this.playHistory : this.favoriteList
			if (list.length === 0) {
				return
			}
			list = list.map((song) => {
				return new Song(song)
			})
			this.randomPlay({list})
		},
		handlePlaylist (playlist) {
			const bottom = playlist.length > 0 ? '60px' : ''
			this.$refs.listWrapper.style.bottom = bottom
			if (!this.currentIndex) {
				this.$refs.favoriteList.refresh()
			} else {
				this.$refs.playList.refresh()
			}
		},
		...mapActions([
				'insertSong',
				'randomPlay'
			])
	},
	computed: {
		noResult () {
			if (this.currentIndex === 0) {
				return !this.favoriteList.length
			} else {
				return !this.playHistory.length
			}
		},
		noResultDesc () {
			if (this.currentIndex === 0) {
				return '暂无收藏歌曲'
			} else {
				return '暂无播放历史'
			}
		},
		...mapGetters([
				'favoriteList',
				'playHistory'
			])
	},
	components: {
		Switches,
		Scroll,
		SongList,
		NoResult
	}
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/variable"

#user-center
  position: fixed
  top: 0
  bottom: 0
  z-index: 100
  width: 100%
  background-color: $color-background
  &.slider-enter-active, &.slider-leave-to
    transition: all 0.3s
  &.slider-enter, &.slider-leave-to
    transform: translate3d(100%, 0, 0)
  .back
    position: absolute
    top: 0
    left: 6px
    z-index: 50
    .icon-back
      display: block
      padding: 10px
      font-size: $font-size-large-x
      color: $color-theme
  .switches-wrapper
    margin: 10px 0 30px 0
  .play-btn
    box-sizing: border-box
    width: 135px
    padding: 7px 0
    margin: 0 auto
    text-align: center
    border: 1px solid $color-text-l
    border-radius: 100px
    font-size: 0
    .icon-play
      display: inline-block
      vertical-align: middle
      margin-right: 6px
      font-size: $font-size-medium-x
    .text
      display: inline-block
      vertical-align: middle
      font-size: $font-size-small
  .list-wrapper
    position: absolute
    top: 110px
    bottom: 0
    width: 100%
    overflow: hidden
    .list-inner
      padding: 20px 30px
  .no-result-wrapper
    position: absolute
    width: 100%
    top: 50%
    transform: translateY(-50%)
</style>