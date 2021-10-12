<template>
	<view class="content">
		<view class="image-div">
			<view class="image-unit" v-for="(item, index) in imageList" :key="item._id">
				<image :src="item.image_url" class="image-card"></image>
				<view class="btn-card"><button class="primary-btn" @click="downLoadImage(item.image_url)">保存头像</button></view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			imageList: [],
			shareInfo: uni.getStorageSync('shareInfo')
		};
	},
	onLoad() {},
	onShareAppMessage: function() {
		return this.shareInfo;
	},
	onShareTimeline: function() {
		return this.shareInfo;
	},
	onLoad() {
		this.getUserImagesByOpenId();
	},
	methods: {
		/**
		 * 获取头像
		 */
		getUserImagesByOpenId() {
			uni.showLoading({
				mask: true
			});
			let { _id: useId } = uni.getStorageSync('user_info');
			uniCloud
				.callFunction({
					name: 'user_images',
					data: { type: 'getUserImagesByOpenId', useId }
				})
				.then(res => {
					this.imageList = res.result.data;
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
		 * 保存图片
		 */
		downLoadImage(imageUrl) {
			//获取相册授权
			let _self = this;
			uni.getSetting({
				success(res) {
					if (!res.authSetting['scope.writePhotosAlbum']) {
						uni.authorize({
							scope: 'scope.writePhotosAlbum',
							success() {
								//这里是用户同意授权后的回调
								_self.saveImgToLocal(imageUrl);
							},
							fail(e) {
								uni.hideLoading();
								wx.showModal({
									content: '检测到您没打开下载图片功能权限，是否去设置打开？',
									confirmText: '确认',
									cancelText: '取消',
									success: function(res) {
										//点击“确认”时打开设置页面
										if (res.confirm) {
											wx.openSetting();
										}
									}
								});
							}
						});
					} else {
						//用户已经授权过了
						_self.saveImgToLocal(imageUrl);
					}
				}
			});
		},
		/**
		 * @param {Object} imageUrl
		 * 保存到相册
		 */
		saveImgToLocal(imageUrl) {
			let _self = this;
			uni.downloadFile({
				url: imageUrl, //图片地址
				success: res => {
					if (res.statusCode === 200) {
						uni.saveImageToPhotosAlbum({
							filePath: res.tempFilePath,
							success: function() {
								uni.hideLoading();
								uni.showToast({
									title: '保存成功',
									icon: 'none'
								});
							},
							fail: function() {
								uni.hideLoading();
								uni.showToast({
									title: '保存失败',
									icon: 'none'
								});
							}
						});
					}
				}
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	display: flex;
	justify-content: space-between;
	padding: 140rpx 100rpx;
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	background-size: 100% 100%;
	background-image: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/0aa778cd-e304-4794-b88d-54f76940ba48.png);
	.image-div {
		height: 85vh;
		overflow-y: auto;
		.image-unit {
			width: 250;
			height: 350rpx;
			background-color: #ffffff;
			box-shadow: 0px 5px 15px 0px rgba(224, 224, 224, 0.4);
			border-radius: 50rpx;
			overflow: hidden;
			.image-card {
				width: 250rpx;
				height: 250rpx;
			}
			.btn-card {
				padding: 10rpx 30rpx 30rpx;
				box-sizing: border-box;
				display: flex;
				.primary-btn {
					width: 150rpx;
					background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
					border: 1rpx solid #ff7852;
					border-radius: 48rpx;
					box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
					color: #fff;
					padding: 0;
					font-size: 24rpx;
					float: right;
				}
			}
		}
	}
}
</style>
