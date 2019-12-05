<template>
	<view class="grace-margin">
		<view class="grace-title grace-margin-top">
			海南阿斯克互联网医院有限公司欢迎您！
		</view>
		<!-- <swiper class="grace-swiper swiper1" autoplay="true" indicator-color="rgba(255, 255, 255, 1)" indicator-active-color="#3688FF"
		 style="height:276rpx" interval="3000">
			<swiper-item>
				<image src='https://askyisheng.oss-cn-hangzhou.aliyuncs.com/sw1.png'></image>
			</swiper-item>
		</swiper> -->
		<form @submit="formSubmit" class="grace-form" style="margin-top:25px;">
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					姓名
				</view>
				<input type="text" name="realname" class="input" placeholder="请输入姓名" />
			</view>
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					省
				</view>
				<input type="text" name="province" class="input" placeholder="请输入省" />
			</view>
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					市
				</view>
				<input type="text" name="city" class="input" placeholder="请输入市" />
			</view>
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					区
				</view>
				<input type="text" name="area" class="input" placeholder="请输入区" />
			</view>
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					手机号
				</view>
				<input type="number" name="phone" class="input" v-model="phone" placeholder="请输入手机号" />
			</view>
			<view class="grace-items grace-noborder" v-if="is_first">
				<view class="grace-label grace-center">验证码</view>
				<input type="number" class="input" name="yzm" placeholder="请输入验证码" />
				<view style="width:180rpx; margin-left:30rpx; flex-shrink:0;">
					<button type="primary" class="login-sendmsg-btn" @tap='getVCode'>{{vcodeBtnName}}</button>
				</view>
			</view>
			<view class="grace-items" v-if="is_first">
				<view class="grace-label grace-center">
					邀请码
				</view>
				<input type="text" name="invite" class="input" placeholder="请输入邀请码" />
			</view>
			<view style="padding:22rpx 0;">
				<button v-if="is_first" formType="submit" type="primary" style="width:100%;">绑定</button>
				<button v-else formType="submit" type="primary" style="width:100%;">登陆</button>
			</view>
		</form>
	</view>
</template>
<script>
	var graceChecker = require("../../graceUI/jsTools/graceChecker.js");
	var graceRequest = require("../../graceUI/jsTools/request.js");
	export default {
		data() {
			return {
				openid: "",
				vcodeBtnName: "获取验证码",
				is_first: false,
				countNum : 120,
				countDownTimer : null,
				phone : ''
			}
		},
		onShow() {
			uni.setStorageSync('drugrepid',1)
			if (!uni.getStorageSync('drugrepid')) {
				let appid = "wx236bc250708f2a2f"; //为测试号id
				let code = getUrlParam("code"); //是否存在code
				let _this = this;
				console.log(code);
				let local = window.location.href;
				
				if (code == null || code === "") {
					//不存在就打开上面的地址进行授权
					window.location.href = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${appid}&redirect_uri=${encodeURIComponent(local)}&response_type=code&scope=snsapi_userinfo&state=1#wechat_redirect`;
				} else {
					//存在则通过code传向后台调用接口返回微信的个人信息
					graceRequest.get(
						'https://askapp.wxori.top/wx/getWXinfo',
						{code:code},
						function(res){
							if(res.code == '0'){
								_this.openid = res.data.openid
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
			} else {
				uni.redirectTo({
					url: '../person/person'
				});
			}
		},
		methods: {
			getVCode: function() {
				var myreg = /^[1][1,2,3,4,5,7,8,9][0-9]{9}$/;
				if (!myreg.test(this.phone)) {
					uni.showToast({
						title: '请正确填写手机号码',
						icon: "none"
					});
					return false;
				}
				// 手机号码为 :  this.phone
				// vcodeBtnName 可以阻止按钮被多次点击 多次发送 return 会终止函数继续运行
				if (this.vcodeBtnName != '获取验证码' && this.vcodeBtnName != '重新发送') {
					return;
				}
				this.vcodeBtnName = "发送中...";
				let data = {}
				data.phone = this.phone
				data.type = 5
				graceRequest.post(
					'https://askapp.wxori.top/index/buildCode',
					data,
					'form', {},
					function(res) {
						uni.showToast({
							title: res.msg,
							icon: 'none',
							mask: true
						});
					}
				);
				// 与后端 api 交互，发送验证码 【自己写的具体业务代码】
				// 假设发送成功，给用户提示
				uni.showToast({
					title: '短信已发送，请注意查收',
					icon: "none"
				});
				// 倒计时
				this.countNum = 120;
				this.countDownTimer = setInterval(function() {
					this.countDown();
				}.bind(this), 1000);
			},
			countDown: function() {
				if (this.countNum < 1) {
					clearInterval(this.countDownTimer);
					this.vcodeBtnName = "重新发送";
					return;
				}
				this.countNum--;
				this.vcodeBtnName = this.countNum + '秒重发';
			},
			loginsuccess: function(e) {
				uni.setStorageSync('drugrepid',e.id)
				uni.redirectTo({
					url: '../person/person'
				});
			},
			formSubmit: function(e) {
				const _this = this
				var formData = e.detail.value;
				if (_this.is_first) {
					//定义表单规则
					var rule = [{
						name: "realname",
						checkType: "notnull",
						checkRule: "",
						errorMsg: "请填写姓名"
					},{
						name: "province",
						checkType: "notnull",
						checkRule: "",
						errorMsg: "请填写省"
					},{
						name: "city",
						checkType: "notnull",
						checkRule: "",
						errorMsg: "请填写市"
					},{
						name: "area",
						checkType: "notnull",
						checkRule: "",
						errorMsg: "请填写区"
					},{
						name: "phone",
						checkType: "phone",
						checkRule: "",
						errorMsg: "请填写正确的手机号"
					},
					{
						name: "yzm",
						checkType: "notnull",
						checkRule: "",
						errorMsg: "请填写验证码"
					}];
					//进行表单检查

					var checkRes = graceChecker.check(formData, rule);
					if (checkRes) {
						if (_this.openid) {
							formData.openid = _this.openid
							console.log(formData);
							graceRequest.post(
								'https://askapp.wxori.top/drugrep/drugrepbind',
								formData,
								'form', {},
								function(res) {
									if(res.code == '0'){
										_this.loginsuccess(res.data)
									}else{
										uni.showToast({
											title: res.msg,
											icon: 'none',
											mask: true
										});
									}
								}
							);
						} else {
							uni.showToast({
								title: '获取用户信息失败，请刷新重试',
								icon: 'none',
								mask: true
							});
						}
					} else {
						uni.showToast({
							title: graceChecker.error,
							mask: true,
							icon: "none",
							duration: 600
						});
					}
				} else {
					if (_this.openid) {
						formData.openid = _this.openid

						console.log(formData);
						graceRequest.post(
							'https://askapp.wxori.top/drugrep/drugreplogin',
							formData,
							'form', {},
							function(res) {
								res.code != '0' && uni.showToast({
									title: res.msg,
									icon: 'none',
									mask: true
								});
								res.code == '0' && _this.loginsuccess(res.data)
								if (res.code == '-2') {
									_this.is_first = true
								}
							}
						);
					} else {
						uni.showToast({
							title: '获取用户信息失败，请刷新重试',
							icon: 'none',
							mask: true
						});
					}
				}
			}
		}
	}
	// 判断公众号截取code
	const getUrlParam = (name) => {
		let reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
		let r = window.location.search.substr(1).match(reg);
		if (r != null) {
			return unescape(r[2]);
		}
		return null;
	}
</script>
<style>
	.swiper1 image {
		width: 690rpx;
		height: 276rpx;
	}

	.swiper1 swiper-item {
		border-radius: 10rpx;
	}
	/* uni-app 中使用 px 可以实现不同设备下字体大小一致， 并非只能使用 rpx */
	.login-sendmsg-btn{border:1px solid #3688FF !important; background:none !important; color:#3688FF !important; width:100%; height:32px; line-height:32px; font-size:12px !important;}
	.grace-login-three{display:flex; justify-content:center; flex-wrap:nowrap;}
	.grace-login-three view{width:50px; height:50px; line-height:50px; font-size:38px; color:#3688FF; text-align:center; margin:10rpx;}
	.marginTop50{margin-top:50px;}
	.marginTop30{margin-top:30px;}
	
	.grace-line-title{display:flex; flex-direction:row; flex-wrap:nowrap; align-items:center;}
	.grace-line-title > .line{width:50rpx; flex:auto; height:1px; background:#F9F9F9;}
	.grace-line-title > .title{padding:0 80rpx; line-height:80rpx;}
</style>
