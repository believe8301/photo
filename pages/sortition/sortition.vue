<template>
	<view class="content" :style="{ 'background-image': 'url(' + indexBg.imageUrl + ')' }">
		<view class="iconfont icon-xiangzuo" @click="navBack"></view>
		<view class="sign-card">
			<view class="sign-div" :style="{ 'background-image': 'url(' + signBg.imageUrl + ')' }" v-if="signState">
				<view class="sign-title">{{ signContent.name }}</view>
				<view class="sign-content">签语：{{ signContent.content }}</view>
			</view>
		</view>
		<view class="btn-card">
			<button class="primary-btn" v-if="userInfo" @click.stop="getSign()">点击获取你的新年签</button>
			<button class="primary-btn" v-else open-type="getUserInfo" @click.stop="getUserProfile('userLogin')">点击获取你的新年签</button>
			<button open-type="share" type="text" class="share-btn" @click.stop>送好友一支新年签</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			signState: false,
			code: '',
			shareInfo: uni.getStorageSync('shareInfo'),
			indexBg: uni.getStorageSync('background_info') ? uni.getStorageSync('background_info').find(el => el.code === 'sortition_bg') : {},
			signBg: uni.getStorageSync('background_info') ? uni.getStorageSync('background_info').find(el => el.code === 'sign_bg') : {},
			signContent: {},
			userInfo: uni.getStorageSync('user_info')
		};
	},
	onShareAppMessage: function() {
		return this.shareInfo;
	},
	onShareTimeline: function() {
		return this.shareInfo;
	},
	onLoad() {
		this.init();
	},
	methods: {
		async init() {
			this.code = await this.getWeixinCode();
			if (this.userInfo) {
				this.getSign('onlyGet');
			}
		},
		/**
		 * 获取微信code
		 */
		getWeixinCode() {
			return new Promise((resolve, reject) => {
				uni.login({
					provider: 'weixin',
					success(res) {
						resolve(res.code);
					},
					fail(err) {
						reject(new Error('微信登录失败'));
					}
				});
			});
		},
		/**
		 * @param {Object} type
		 * 登录
		 */
		async getUserProfile(type) {
			let _this = this;
			uni.getUserProfile({
				desc: '获取您的头像信息',
				success(result) {
					let data = {
						code: _this.code,
						signature: result.signature,
						encrypted_data: result.encryptedData,
						iv: result.iv,
						userInfo: result.userInfo
					};
					_this.postUserInfo(result.userInfo.nickName);
				},
				fail(fall) {}
			});
		},
		/**
		 * @param {Object} nickName
		 * 存储用户数据
		 */
		async postUserInfo(nickName) {
			let that = this;
			this.code = await this.getWeixinCode();
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			uniCloud
				.callFunction({
					name: 'user_mpweixin',
					data: { code: that.code, nickName, type: 'userLogin' }
				})
				.then(res => {
					uni.setStorageSync('user_info', res.result);
					this.userInfo = res.result;
					this.getSign();
				})
				.finally(() => {
					uni.hideLoading();
				});
		},
		getSign(setType) {
			if (this.signState) {
				uni.showToast({
					icon: 'none',
					title: '签语虽好，不可贪多哦~'
				});
				return;
			}
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			let { _id: userId } = uni.getStorageSync('user_info');
			uniCloud
				.callFunction({
					name: 'sign',
					data: { type: 'drawLots', userId, setType }
				})
				.then(res => {
					if (res.result && res.result.name) {
						this.signState = true;
						this.signContent = res.result;
					}
				})
				.catch(err => {
					uni.showModal({
						content: err.message || '请求服务失败',
						showCancel: false
					});
				})
				.finally(() => {
					uni.hideLoading();
				});
		},
		/**
		 * 返回上一页
		 */
		navBack() {
			uni.navigateBack({
				delta: 1
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	// padding-top: 160rpx;
	height: 100vh;
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	background-size: 100% 100%;
	.icon-xiangzuo {
		position: fixed;
		top: 6vh;
		left: 30rpx;
		color: #ffffff;
		font-size: 40rpx;
		font-weight: bold;
	}
	.sign-card {
		width: 350rpx;
		height: 530rpx;
		margin: 80rpx auto 50rpx;
		.sign-div {
			padding: 30rpx;
			height: 100%;
			width: 100%;
			background-size: 100% 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			flex-direction: column;
			animation: turnY 3s ease-out;
			.sign-title {
				padding-bottom: 20rpx;
				font-size: 50rpx;
				font-weight: bold;
				background-image: -webkit-linear-gradient(90deg, #ffa462, #ff4d42 88.36%);
				-webkit-text-fill-color: transparent;
				-webkit-background-clip: text;
				animation: fade-in 3s ease-out;
			}
			.sign-content {
				font-size: 30rpx;
				color: #ff4d42;
				animation: fade-in 3s ease-out;
			}
		}
	}
	.btn-card {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		padding: 50rpx;
		.primary-btn {
			display: inline-block;
			background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
			border: 1rpx solid #ff7852;
			border-radius: 40rpx;
			box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
			color: #fff;
			width: 400rpx;
			font-size: 30rpx;
			height: 80rpx;
			line-height: 78rpx;
		}
		.share-btn {
			background-color: transparent;
			border-width: 0;
			font-size: 30rpx;
			color: #ffa462;
		}
		.share-btn::after {
			border-left-width: 0;
			border-right-width: 0;
			border-top-width: 0;
			border-bottom-width: 0;
		}
	}
}
@keyframes turnY {
	0% {
		transform: rotateY(0deg);
	}
	100% {
		transform: rotateY(1080deg);
	}
}
@keyframes fade-in {
	0% {
		opacity: 0;
	}
	100% {
		opacity: 1;
	}
}
</style>
