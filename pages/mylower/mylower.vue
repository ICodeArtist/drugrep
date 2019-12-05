<template>
	<view>xj
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			}
		},
		onShow() {
			const _this = this
			if (!uni.getStorageSync('drugrepid')) {
				uni.redirectTo({
					url: '../index/index'
				});
			}else{
				
			}
		},
		methods: {
			
		}
	}
</script>

<style>

</style>
