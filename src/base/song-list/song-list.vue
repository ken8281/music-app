<template>
	<div id="song-list">
		<ul>
			<li @click='selectItem(song, index, $event)' class='item' v-for='(song, index) in songs'>
        <div v-if='rank' class="rank">
          <span :class='getRankCls(index)'>{{getRankText(index)}}</span>
        </div>
				<div class="content">
					<h2 class="name">{{song.name}}</h2>
					<p class="desc">{{song | getDesc}}</p>
				</div>
			</li>
		</ul>
	</div>
</template>

<script type="text/ecmascript-6">
export default {
	props: {
		songs: {
			type: Array,
			default: []
		},
    rank: {
      type: Boolean,
      default: false
    }
	},
  methods: {
    selectItem (item, index, e) {
      if (!e._constructed) {
        return
      }
      this.$emit('select', item, index)
    },
    getRankCls(index) {
      if(index <= 2) {
        return `icon icon${index}`
      } else {
        return 'text'
      }
    },
    getRankText(index) {
      if(index > 2) {
        return index + 1
      }
    }
  },
  // computed: {
  //   getRankCls() {
  //     let arr = []
  //     for(let i = 0; i < this.songs.length; i++) {
  //       if(i <= 2) {
  //         arr.push(`icon icon${i}`)
  //       } else {
  //         arr.push('text')
  //       }
  //     }
  //     return arr
  //   },
  //   getRankText () {
  //     let arr = []
  //     for(let i = 0; i < this.songs.length; i++) {
  //       if(i <= 2) {
  //         arr.push('')
  //       } else {
  //         arr.push(i + 1)
  //       }
  //     }
  //     return arr
  //   }
  // },
	filters: {
		getDesc(song) {
			return `${song.singer}·${song.album}`
		}
	}
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import "~common/stylus/variable"
@import "~common/stylus/mixin"

#song-list
  .item
    display: flex
    align-items: center
    box-sizing: border-box
    height: 64px
    font-size: $font-size-medium
    .rank
      flex: 0 0 25px
      width: 25px
      margin-right: 30px
      text-align: center
      .icon
        display: inline-block
        width: 25px
        height: 24px
        background-size: 25px 24px
        &.icon0
          bg-image('first')
        &.icon1
          bg-image('second')
        &.icon2
          bg-image('third')
      .text
        color: $color-theme
        font-size: $font-size-large
    .content
      flex: 1
      line-height: 20px
      overflow: hidden
      .name
        no-wrap()
        color: $color-text
      .desc
        no-wrap()
        margin-top: 4px
        color: $color-text-d
</style>