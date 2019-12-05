<template>
	<view>
		<view class="grace-margin-top">
			<text class="grace-title grace-margin-left">点击复制客服QQ号,添加好友后咨询问题</text>
		</view>
		<view class="grace-list grace-body">
			<view class="items" @click="showDialog(answer.qq)" v-for="(answer,index) in answers" v-bind:key="index">
				<view class="icons grace-icons icon-kf3 grace-red"></view>
				<view class="body">
					<view class="title">{{answer.qq}}</view>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
var graceRequest = require("../../graceUI/jsTools/request.js");
import '../../js_sdk/ican-H5Api/ican-H5Api.js';
export default {
	data() {
		return {
			answers:[]
		}
	},
	onLoad(options){
		const _this = this
		uni.showLoading({});
		graceRequest.get(
			'https://askapp.wxori.top/back/answerqq',
			{type:options.type},
			function(res){
				if(res.code == '0'){
					console.log(res);
					setTimeout(() => {
						_this.answers = res.data
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
	},
	methods:{
		showDialog(e){
			console.log(e);
			const _this = this
			uni.setClipboardData({ 
				data:e, 
				success:function(data){
					console.log('success');
					_this.show1 = false;
					uni.showToast({
					    title: '复制成功',
						icon:'success',
						mask:true
					});
				}, 
				fail:function(err){
					console.log('fail');
				}, 
				complete:function(res){
					console.log('complete');
				} ,
			})
		}
	}
}
</script>
<style>
.icons{margin-top: 15rpx;}
.demo2{height:60rpx; width:200rpx;}
</style>