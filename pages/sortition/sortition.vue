<template>
	<view class="content">
		<view class="iconfont icon-xiangzuo" @click="navBack"></view>
		<view class="sign-div">
			<view class="sign-title">暴富签</view>
			<view class="sign-content">签语：今年报复</view>
		</view>
		<view class="btn-card">
			<button class="primary-btn" @click.stop="navOriginal()">点击获取你的新年签</button>
			<button open-type="share" type="text" class="share-btn" @click.stop>送好友一支新年签</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {};
	},
	methods: {
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
	padding-top: 160rpx;
	height: 100vh;
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background-size: 100% 100%;
	background-image: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/65475d31-c22e-4dcb-85bd-6d3f9099d860.jpg);
	.icon-xiangzuo {
		position: fixed;
		top: 6vh;
		left: 30rpx;
		color: #ffa462;
		font-size: 40rpx;
		font-weight: bold;
	}
	.sign-div {
		background-color: #ffffff;
		width: 300rpx;
		margin: 80rpx auto 0;
		padding: 30rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		.sign-title {
			writing-mode: vertical-lr;
			padding-bottom: 20rpx;
			font-size: 50rpx;
			font-weight: bold;
			background-image: -webkit-linear-gradient(90deg, #ffa462, #ff4d42 88.36%);
			-webkit-text-fill-color: transparent;
			-webkit-background-clip: text;
		}
		.sign-content {
			font-size: 30rpx;
			color: #ff4d42;
		}
	}
	.btn-card {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		padding: 50rpx;
		position: fixed;
		left: 125rpx;
		bottom: 300rpx;
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
			color: #ff4d42;
		}
		.share-btn::after {
			border-left-width: 0;
			border-right-width: 0;
			border-top-width: 0;
			border-bottom-width: 0;
		}
	}
}
</style>
