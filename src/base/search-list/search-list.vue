<template>
	<div id="search-list" v-show='searches.length'>
		<transition-group name='list' tag='ul'>
			<li @click='selectItem(item, $event)' v-for='item in searches' :key='item' class="search-item">
				<span class="text">{{item}}</span>
				<span @click.stop='deleteOne(item, $event)' class="icon">
					<i class="icon-delete"></i>
				</span>
			</li>
		</transition-group>
	</div>
</template>

<script type='text/ecmascript-6'>
export default {
	props: {
		searches: {
			type: Array,
			default: []
		}
	},
	methods: {
		selectItem(item, e) {
			this.$emit('select', item, e)
		},
		deleteOne(item, e) {
			this.$emit('delete', item, e)
		}
	}
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import "~common/stylus/variable"
@import '~common/stylus/mixin'

#search-list
  .search-item
    display: flex
    align-items: center
    height: 40px
    overflow: hidden
    &.list-enter-active, &.list-leave-active
      transform: all 0.1s
    &.list-enter, &.list-leave-to
      height: 0
    .text
      flex: 1
      color: $color-text-l
    .icon
      extend-click()
      .icon-delete
        font-size: $font-size-small
        color: $color-text-d
</style>