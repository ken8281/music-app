<template>
	<div id="progress-bar" ref='progressBar' @click='progressClick'>
		<div class="bar-inner">
			<div class="progress" ref='progress'></div>
			<div @touchstart.prevent='progressTouchStart'
			 @touchmove.prevent='progressTouchMove'
			 @touchend='progressTouchEnd'
			 class="progress-btn-wrapper"
			  ref='progressBtn'>
				<div class="progress-btn"></div>
			</div>
		</div>
	</div>
</template>

<script type='text/ecmascript-6'>
import {prefixStyle} from 'common/js/dom'

const progressBtnWidth = 16
const transform = prefixStyle('transform')

export default {
	created () {
		this.touch = {}
	},
	props: {
		percent: {
			type: Number,
			default: 0
		}
	},
	methods: {
		progressTouchStart (e) {
			this.touch.initiated = true
			this.touch.startX = e.touches[0].pageX
			this.touch.left = this.$refs.progress.clientWidth
		},
		progressTouchMove (e) {
			if (!this.touch.initiated) {
				return
			}
			const barWidth = this.$refs.progressBar.clientWidth - progressBtnWidth
			let delta = e.touches[0].pageX - this.touch.startX
			let offsetWidth = Math.min(Math.max(0, this.touch.left + delta), barWidth)
			this._offset(offsetWidth)
		},
		progressTouchEnd () {
			this.touch.initiated = false
			this._triggerPercent()
		},
		progressClick (e) {
			const left = this.$refs.progressBar.getBoundingClientRect().left
			this._offset(e.pageX - left)
			this._triggerPercent()
		},
		setProgressOffset(percent) {
	      if (percent >= 0 && !this.touch.initiated) {
	        const barWidth = this.$refs.progressBar.clientWidth - progressBtnWidth
	        const offsetWidth = percent * barWidth
	        this._offset(offsetWidth)
	      }
	    },
		_triggerPercent() {
			const barWidth = this.$refs.progressBar.clientWidth - progressBtnWidth
			const offsetWidth = this.$refs.progress.clientWidth
			const percent = offsetWidth / barWidth
			this.$emit('percentchange', percent)
		},
		_offset(offsetWidth) {
			this.$refs.progress.style.width = `${offsetWidth}px`
			this.$refs.progressBtn.style[transform] = `translate3d(${offsetWidth}px, 0, 0)`
		}
	},
	watch: {
		percent(val) {
			this.setProgressOffset(val)
		}
	}
}
</script>

<style scoped lang='stylus' rel="stylesheet/stylus">
@import "~common/stylus/variable"

#progress-bar
  height: 30px
  .bar-inner
    position: relative
    top: 13px
    height: 4px
    background-color: rgba(0, 0, 0, 0.3)
    .progress
      position: absolute
      height: 100%
      background-color: $color-theme
    .progress-btn-wrapper
      position: absolute
      left: -8px
      top: -13px
      width: 30px
      height: 30px
      .progress-btn
        position: relative
        top: 7px
        left: 7px
        box-sizing: border-box
        width: 16px
        height: 16px
        border: 3px solid $color-text
        border-radius: 50%
        background-color: $color-theme
</style>