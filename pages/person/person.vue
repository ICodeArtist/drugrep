<template>
	<view>
		<view class="grace-body">
			<view class="grace-list" style="margin-top:10rpx; margin-bottom:30rpx;">
				<view class="items">
					<view class="body">
						<view :class="['title', graceSkeleton == 'ing' ? 'grace-skeleton' : '']">{{realname}}<text :class="[graceSkeleton == 'ing' ? 'grace-skeleton' : 'grace-badge grace-bg-red']">{{levelname}}</text></view>
						<view :class="['desc', graceSkeleton == 'ing' ? 'grace-skeleton' : '']">{{comefrom}}</view>
					</view>
				</view>
			</view>
		</view>
		<view class="ucenter-line"></view>
		<view class="grace-box-banner">
			<navigator url="../mylower/mylower" class="garce-items">
				<view class="line1">{{one_invite}}</view>
				<view class="line2">我的下级</view>
			</navigator>
			<!-- <view class="garce-items">
				<view class="line1">29</view>
				<view class="line2">好友</view>
			</view> -->
		</view>
		<view class="ucenter-line"></view>
		<view class="grace-list grace-body">
			<view class="items" @click="showDialog1">
				<view class="icons grace-icons icon-share2 grace-blue"></view>
				<view class="body">
					<view class="title">我的邀请码</view>
				</view>
				<view class="arrow-right"></view>
			</view>
			<navigator url="../answer/answer?type=4" class="items">
				<view class="icons grace-icons icon-kf3 grace-red"></view>
				<view class="body">
					<view class="title">客服qq</view>
				</view>
				<view class="arrow-right"></view>
			</navigator>
		</view>
		<!-- <view class="ucenter-line"></view> -->
		<!-- <view class="grace-list grace-body">
			<navigator class="items">
				<view class="icons grace-icons icon-set grace-yellow"></view>
				<view class="body">
					<view class="title">账户设置</view>
				</view>
				<view class="arrow-right"></view>
			</navigator>
			<navigator class="items">
				<view class="icons grace-icons icon-phone grace-red"></view>
				<view class="body">
					<view class="title">手机密保</view>
				</view>
				<view class="arrow-right"></view>
			</navigator>
		</view> -->
		<view class="ucenter-line"></view>
		<graceDialog title="我的邀请码" :show="show1" :isBtns="true" v-on:closeDialog="closeDialog1">
			<view slot="content">
				<view class="content1">{{invite_code}}</view>
			</view>
			<view slot="btns">
				<view class="grace-dialog-btns">
					<view><button type="primary" @tap="closeDialog1">关闭</button></view>
					<view><button type="primary" style="color:#3688FF;" @tap="copycode">复制</button></view>
				</view>
			</view>
		</graceDialog>
	</view>
</template>
<script>
	var graceRequest = require("../../graceUI/jsTools/request.js");
	import graceDialog from '../../graceUI/components/graceDialog.vue';
	import '../../js_sdk/ican-H5Api/ican-H5Api.js';
	export default {
		components : {
				graceDialog : graceDialog
			},
		data() {
			return {
				graceSkeleton : 'ing',
				realname: '',
				levelname: '',
				comefrom: '',
				level: '0',
				one_invite: '0',
				invite_code: '',
				show1: false
			}
		},
		onShow() {
			const _this = this
			if (!uni.getStorageSync('drugrepid')) {
				uni.redirectTo({
					url: '../index/index'
				});
			}else{
				uni.showLoading({});
				graceRequest.get(
					'https://askapp.cloudhos.net/drugrep/getRepInfo',
					{drugrepid:uni.getStorageSync('drugrepid')},
					function(res){
						if(res.code == '0'){
							setTimeout(() => {
								_this.realname = res.data.realname;
								_this.levelname = res.data.levelname;
								_this.level = res.data.level;
								_this.one_invite = res.data.one_invite;
								_this.invite_code = res.data.invite_code;
								_this.comefrom = res.data.province + '-' + res.data.city + '-' + res.data.area;
								_this.graceSkeleton = 'end';
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
			showDialog1 : function(){
				if(this.level == '0'){
					uni.showToast({
					    title: '未签约',
						icon:'none',
						mask:true
					});
				}else{
					this.show1 = true;
				}
			},
			closeDialog1 : function(){
				this.show1 = false;
			},
			copycode : function() {
				const _this = this
				uni.setClipboardData({ 
					data:this.invite_code, 
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
.ucenter-face{width:120rpx !important; height:120rpx !important;}
.ucenter-face image{width:120rpx !important; height:120rpx !important;}
.ucenter-line{width:100%; height:10px; background:#F4F5F6; margin:8px 0;}
.content1{padding:20rpx; font-size:26rpx; text-align:center; line-height:60rpx; background:#F8F8F8; border-bottom-left-radius:10rpx; border-bottom-right-radius:10rpx;}
</style>