<template>
	<view>
		<view class="grace-list grace-body" v-if="lowers !== null && lowers.length>0">
			<view class="items" v-for="(lower,index) in lowers" v-bind:key="index">
				<view class="body">
					<view class="title">{{lower.realname}}<text :class="[lower.level == '0' ? '' : 'grace-red']">{{lower.levelname}}</text></view>
				</view>
			</view>
		</view>
		<view v-else style="padding:50% 0;">
			<graceEmpty text="暂无数据" :iconSize="80" :iconType="2" iconColor="#3688FF"></graceEmpty>
		</view>
	</view>
</template>
<script>
var graceRequest = require("../../graceUI/jsTools/request.js");
import graceEmpty from "../../graceUI/components/graceEmpty.vue";
export default {
	data() {
		return {
			lowers:[]
		}
	},
	onShow(){
		const _this = this
		if (!uni.getStorageSync('drugrepid')) {
			uni.redirectTo({
				url: '../index/index'
			});
		}else{
			uni.showLoading({});
			graceRequest.get(
				'https://askapp.wxori.top/drugrep/mylower',
				{drugrepid:uni.getStorageSync('drugrepid')},
				function(res){
					if(res.code == '0'){
						console.log(res);
						setTimeout(() => {
							_this.lowers = res.data
							uni.hideLoading();
						}, 800);
					}else{
						uni.showToast({
						    title: res.msg,
							icon:'none',
							mask:true
						});
					}
				}
			);
		}
	},
	methods:{

	},
	components: {
			graceEmpty
		}
}
</script>
<style>
.icons{margin-top: 15rpx;}
.demo2{height:60rpx; width:200rpx;}
</style>