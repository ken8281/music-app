<template>
	<div id="search">
		<div class='search-box-wrapper'>
			<search-box @query='onQueryChange' ref='searchBox'/>
		</div>
		<div ref='shortcutWrapper' v-show='!query' class="shortcut-wrapper">
			<scroll :data='shortcut' ref='shortcut' class="shortcut">
				<div>
					<div class="hot-key">
						<h1 class="title">热门搜索</h1>
						<ul>
							<li @click='addQuery(item.k, $event)' v-for='item in hotKey' class="item">
								<span>{{item.k}}</span>
							</li>
						</ul>
					</div>
					<div v-show='searchHistory.length' class="search-history">
						<h1 class="title">
							<span class="text">搜索历史</span>
							<span @click='showConfirm' class="clear">
								<i class="icon-clear"></i>
							</span>
						</h1>
						<search-list @select='addQuery' @delete='deleteOne' :searches='searchHistory'/>
					</div>
				</div>
			</scroll>
		</div>
		<div ref='searchResult' v-show='query' class="search-result">
			<suggest @select='saveSearch' @listScroll='blurInput' ref="suggest" :query='query'/>
		</div>
		<confirm text='是否清空所有搜索历史' confirmBtnText='清空' @confirm='clearSearchHistory' ref='confirm'/>
		<router-view/>
	</div>
</template>

<script type='text/ecmascript-6'>
import SearchBox from 'base/search-box/search-box'
import {getHotKey} from 'api/search'
import {ERR_OK} from 'api/config'
import Suggest from 'components/suggest/suggest'
import {mapActions} from 'vuex'
// import {mapActions, mapGetters} from 'vuex'
import SearchList from 'base/search-list/search-list'
import Confirm from 'base/confirm/confirm'
import Scroll from 'base/scroll/scroll'
import {playlistMixin,searchMixin} from 'common/js/mixin'

export default {
	mixins: [playlistMixin,searchMixin],
	data () {
		return {
			hotKey: [],
			// query: ''
		}
	},
	created () {
		this._getHotKey()
	},
	computed: {
		shortcut () {
			return this.hotKey.concat(this.searchHistory)
		},
		// ...mapGetters([
		// 		'searchHistory'
		// 	])
	},
	methods: {
		// addQuery (query, e) {
		// 桌面端双点击，在mixin处理
		// 	this.$refs.searchBox.setQuery(query)
		// },
		// onQueryChange (query) {
		// 	this.query = query
		// },
		// blurInput () {
		// 	this.$refs.searchBox.blur()
		// },
		// saveSearch () {
		// 	this.saveSearchHistory(this.query)
		// },
		// deleteOne (item, e) {
		// 	if (!e._constructed) {
		//         return
		//     }
		// 	this.deleteSearchHistory(item)
		// },
		showConfirm (e) {
			if (!e._constructed) {
		        return
		    }
			this.$refs.confirm.show()
		},
		handlePlaylist (playlist) {
			const bottom = playlist.length ? '60px' : ''
			this.$refs.shortcutWrapper.style.bottom = bottom
			this.$refs.shortcut.refresh()

			this.$refs.searchResult.style.bottom = bottom
			this.$refs.suggest.refresh()
		},
		_getHotKey () {
			getHotKey().then((res) => {
				if (res.code === ERR_OK) {
					this.hotKey = res.data.hotkey.slice(0, 10)
				}
			})
		},
		...mapActions([
				// 'saveSearchHistory',
				// 'deleteSearchHistory',
				'clearSearchHistory'
			])
	},
	watch: {
		query (val) {
			if (!val) {
				this.$nextTick(() => {
					this.$refs.shortcut.refresh()
				})
			}
		}
	},
	components: {
		SearchBox,
		Suggest,
		SearchList,
		Confirm,
		Scroll
	}
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import '~common/stylus/variable';
@import '~common/stylus/mixin';

#search
  .search-box-wrapper
    margin: 20px
  .shortcut-wrapper
    position: fixed
    top: 178px
    bottom: 0
    width: 100%
    .shortcut
      height: 100%
      overflow: hidden
      .hot-key
        margin: 0 20px 20px 20px
        .title
          margin-bottom: 20px
          font-size: $font-size-medium
          color: $color-text-l
        .item
          display: inline-block
          padding: 5px 10px
          margin: 0 20px 10px 0
          border-radius: 6px
          background-color: $color-highlight-background
          font-size: $font-size-medium
          color: $color-text-d
      .search-history
        position: relative
        margin: 0 20px
        .title
          display: flex
          align-items: center
          height: 40px
          font-size: $font-size-medium
          color: $color-text-l
          .text
            flex: 1
          .clear
            extend-click()
            .icon-clear
              font-size: $font-size-medium
              color: $color-text-d
  .search-result
    position: fixed
    width: 100%
    top: 178px
    bottom: 0
</style>