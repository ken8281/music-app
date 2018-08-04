<template>
	<div id="recommend" ref='recommend'>
		<scroll ref='scroll' :data='discList' class="recommend-content">
			<div>
				<div v-if="recommends.length" class="slider-wrapper" ref="sliderWrapper">
		          <slider ref="slider">
		            <div v-for="item in recommends">
		              <a :href="item.linkUrl">
		                <img class='needsclick' @load.once='loadImage' :src="item.picUrl">
		              </a>
		            </div>
		          </slider>
		        </div>
				<div class="recommend-list">
					<h1 class="list-title">热门歌单推荐</h1>
					<ul>
						<li @click='selectItem(item, $event)' v-for='item in discList' class='item'>
							<div class="icon">
								<img height='60' width='60' v-lazy="item.imgurl">
							</div>
							<div class="text">
								<h2 class="name">{{item.creator.name}}</h2>
								<p class='desc'>{{item.dissname}}</p>
							</div>
						</li>
					</ul>
				</div>
			</div>
			<div class="loading-container" v-if='!discList.length'>
				<loading/>
			</div>
		</scroll>
		<router-view/>
	</div>
</template>

<script type='text/ecmascript-6'>
import {ERR_OK} from 'api/config'
import {getRecommend, getDiscList} from 'api/recommend'
import Slider from 'base/slider/slider'
import Scroll from 'base/scroll/scroll'
import Loading from 'base/loading/loading'
import {playlistMixin} from 'common/js/mixin'
import {mapMutations} from 'vuex'

export default {
	mixins: [playlistMixin],
	data () {
		return {
			recommends: [],
			discList: []
		}
	},
	created () {
		this._getRecommend()
		this._getDiscList()
	},
	activated() {
      this.$nextTick(() => {
        this.$refs.slider && this.$refs.slider.refresh()
      })
    },
	methods: {
		selectItem (item, e) {
			if (!e._constructed) {
		        return
		    }
			this.$router.push({
				path: `/recommend/${item.dissid}`
			})
			this.setDisc(item)
		},
		handlePlaylist (playlist) {
	      const bottom = playlist.length ? '60px' : ''
	      this.$refs.recommend.style.bottom = bottom
	      this.$refs.scroll.refresh()
	    },
		_getRecommend () {
			getRecommend().then((res) => {
				if (res.code === ERR_OK) {
					this.recommends = res.data.slider;
				}
			})
		},
		_getDiscList () {
			getDiscList().then((res) => {
				if (res.code ===ERR_OK) {
					this.discList = res.data.list;
				}
			})
		},
		loadImage () {
			this.$refs.scroll.refresh();
		},
		...mapMutations({
			setDisc: 'SET_DISC'
		})
	},
	components: {
		Slider,
		Scroll,
		Loading
	}
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import '~common/stylus/variable';

#recommend
  position: fixed
  width: 100%
  top: 88px
  bottom: 0
  .recommend-content
    height: 100%
    overflow: hidden
    .slider-wrapper
      position: relative
      width: 100%
      overflow: hidden
    .recommend-list
      .list-title
        height: 65px
        line-height: 65px
        text-align: center
        font-size: $font-size-medium
        color: $color-theme
      .item
        display: flex
        box-sizing: border-box
        align-items: center
        padding: 0 20px 20px 20px
        .icon
          flex: 0 0 60px
          width: 60px
          padding-right: 20px
		.text
		  display: flex
		  flex-direction: column
		  justify-content: center
		  flex: 1
		  line-height: 20px
		  overflow: hidden
		  font-size: $font-size-medium
		  .name
		    margin-bottom: 10px
		    color: $color-text
		  .desc
		    color: $color-text-d
	.loading-container
	  position: absolute
	  width: 100%
	  top: 50%
	  transform: translateY(-50%)
</style>