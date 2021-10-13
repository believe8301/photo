<template>
	<view class="content">
		<view class="image-div">
			<view class="icon-xiazai iconfont"></view>
			<view class="tip-div">恭喜你，头像已保存到相册</view>
			<image :src="currentImage" class="image-card" mode="widthFix"></image>
		</view>
		<view class="btn-card">
			<button open-type="share" class="primary-btn">发给朋友</button>
			<button class="action-btn" @click="navIndex()">再来一张</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentImage: '',
			shareInfo: uni.getStorageSync('shareInfo')
		};
	},
	onLoad() {
		this.currentImage = uni.getStorageSync('currentImage');
		uni.removeStorageSync('currentImage');
	},
	onShareAppMessage: function() {
		return this.shareInfo;
	},
	onShareTimeline: function() {
		return this.shareInfo;
	},
	methods: {
		/**
		 * 跳到首页
		 */
		navIndex() {
			uni.reLaunch({
				url: '/pages/index/index'
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	padding-top: 160rpx;
	height: 100vh;
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background-size: 100% 100%;
	background-image: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/618f991c-21c7-43b1-8e57-d9c8f64c734c.png);
	.image-div {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		.icon-xiazai {
			color:  #ffa462;
			font-size: 120rpx;
			margin-bottom: 40rpx;
		}
		.tip-div {
			background-color: #ffa462;
			color: #8b4513;
			padding: 8rpx 30rpx;
			border-radius: 30rpx;
			font-size: 30rpx;
			margin-bottom: 40rpx;
		}
		.image-card {
			width: 340rpx;
			border-radius: 20rpx;
			border: 8rpx solid #f8f8f8;
			box-shadow: 0px -5px 15px 0px rgba(224, 224, 224, 0.4);
		}
	}
	.btn-card {
		padding: 10rpx 30rpx 30rpx;
		box-sizing: border-box;
		display: flex;
		margin-top: 80rpx;
		.action-btn {
			width: 320rpx;
			background: #fff;
			border: 1rpx solid #efefef;
			border-radius: 48rpx;
			box-shadow: 0 12rpx 16rpx -8rpx rgba(0, 0, 0, 0.1);
			color: #4d4d4d;
			padding: 0;
			font-size: 36rpx;
			font-weight: bold;
		}
		.primary-btn {
			width: 320rpx;
			background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
			border: 1rpx solid #ff7852;
			border-radius: 48rpx;
			box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
			color: #fff;
			padding: 0;
			font-size: 36rpx;
			font-weight: bold;
		}
	}
}
</style>
