<template>
	<view class="content">
		<image src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/3da3f4b5-271b-48f5-b13f-acb0d362ed91.jpg" class="all-back"></image>
		<view class="top-content">
			<view class="top-title">
				<view class="title-unit" :class="{ 'title-select': item.selected }" v-for="(item, index) in categoriesList" :key="item._id" @click="switchCategory(item)">
					{{ item.name }}
				</view>
			</view>
			<scroll-view scroll-x :show-scrollbar="false" class="scroll-view">
				<view class="image-div">
					<view :class="{ 'image-margin': index !== 0 }" v-for="(info, index) in imageList" :key="info._id" @click="imageClick(info)">
						<image :src="info.image_url"></image>
					</view>
				</view>
			</scroll-view>
		</view>

		<view class="image-card">
			<view class="photo-main-view">
				<view class="avatar-div " id="avatar-container" @touchstart="touchStart" @touchend="touchEnd" @touchmove="touchMove">
					<image class="img" id="avatar-img" :src="avatarImage || defaultImage" @touchstart="touchAvatarBg"></image>
					<!-- <view class="empty-view " v-if="!avatarImage" @click.native="touchAvatarBg" @touchstart="touchAvatarBg">
						<image class="empty" src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/c33782ca-cd2f-4bfc-84eb-0713c52f522f.svg"></image>
					</view> -->
					<image
						class="avatar-default"
						:class="{ 'avatar-border': showBorder }"
						:style="{
							top: maskCenterY - maskSize / 2 - 2 + 'px',
							left: maskCenterX - maskSize / 2 - 2 + 'px',
							transform: 'rotate(' + rotate + 'deg)' + 'scale(' + scale + ')'
						}"
						id="mask-img"
						:src="currentImage.image_url"
						v-if="currentImage && currentImage.image_url"
					></image>
					<view class="drag-div" :style="{ top: handleCenterY - 10 + 'px', left: handleCenterX - 10 + 'px' }" v-show="showBorder">
						<image class="drag-img" id="drag-img" src="/static/images/drag.svg"></image>
					</view>
				</view>

				<view class="ctlbtn">
					<view class="icon-div btn-margin">
						<view class="icon-zuo iconfont" v-if="showSwitch(-1)" @click="switchAvatar(-1)"></view>
						<view class="icon-you iconfont" v-if="showSwitch(1)" @click="switchAvatar(1)"></view>
					</view>
					<button v-if="userInfo" class="action-btn btn-margin" @click="getUserProfile('createImages')">获取头像</button>
					<button class="action-btn btn-margin" v-else open-type="getUserInfo" @click="getUserProfile('createImages')">获取头像</button>
					<button class="action-btn btn-primary" @click="shareFc()">保存头像</button>
				</view>
			</view>
		</view>
		<view class="btn-card">
			<button v-if="userInfo" class="primary-btn" @click="navOriginal()">原头像</button>
			<button class="primary-btn" v-else open-type="getUserInfo" @click="getUserProfile('userLogin')">原头像</button>
			<button open-type="share" class="share-btn">发给朋友</button>
		</view>
		<image :src="posterImage" style="width: 50px;height: 50px;"></image>
		<view class="hideCanvasView">
			<canvas
				class="hideCanvas"
				id="default_PosterCanvasId"
				canvas-id="default_PosterCanvasId"
				:style="{ width: (poster.width || 10) + 'px', height: (poster.height || 10) + 'px' }"
			></canvas>
		</view>
	</view>
</template>

<script>
import _app from '@/util/QS-SharePoster/app.js';
import { getSharePoster } from '@/util/QS-SharePoster/QS-SharePoster.js';
export default {
	data() {
		return {
			poster: {},
			posterImage: '',
			canvasId: 'default_PosterCanvasId',
			userInfo: '',
			code: '',
			avatarImage: uni.getStorageSync('avatar_image'),
			defaultImage: 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/c33782ca-cd2f-4bfc-84eb-0713c52f522f.svg',
			currentImage: {},
			currentIndex: 0,
			imageList: [],
			categoriesList: [],
			shareInfo: {},
			showBorder: true, // 默认是否显示边框
			maskCenterX: uni.upx2px(380) / 2,
			maskCenterY: uni.upx2px(380) / 2,
			handleCenterX: uni.upx2px(360),
			handleCenterY: uni.upx2px(360),
			maskSize: uni.upx2px(380),
			scale: 1,
			rotate: 0,
			mask_center_x: uni.upx2px(380) / 2,
			mask_center_y: uni.upx2px(380) / 2,
			handle_center_x: uni.upx2px(360),
			handle_center_y: uni.upx2px(360),
			scaleCurrent: 1,
			rotateCurrent: 0,
			touch_target: '',
			start_x: 0,
			start_y: 0,
			cansWidth: 380, // 宽度 px
			cansHeight: 380 // 高度 px
		};
	},
	onLoad() {
		this.init();
		this.getCategoriesList();
		this.getShareInfo();
	},
	onReady() {
		var query = wx.createSelectorQuery();
		query.select('.avatar-div').boundingClientRect();
		query.exec(res => {});
	},
	onShareAppMessage: function() {
		return this.shareInfo;
	},
	onShareTimeline: function() {
		return this.shareInfo;
	},
	methods: {
		/**
		 * 获取分享字典
		 */
		getShareInfo() {
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			uniCloud
				.callFunction({
					name: 'code-mag',
					data: { type: 'mpweixinGet' }
				})
				.then(res => {
					if (res.result.data && res.result.data.length > 0) {
						this.shareInfo = { ...res.result.data[0], imageUrl: res.result.data[0].image_url };
						uni.setStorageSync('shareInfo', this.shareInfo);
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
		 * 获取分类
		 */
		getCategoriesList() {
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			uniCloud
				.callFunction({
					name: 'categories',
					data: { type: 'mpweixin' }
				})
				.then(res => {
					this.categoriesList = res.result.data;
					if (this.categoriesList.length > 0) {
						this.$set(this.categoriesList[0], 'selected', true);
						this.getImagesList(this.categoriesList[0]._id);
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
		 * @param {Object} id
		 * 获取头像
		 */
		getImagesList(id, num) {
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			uniCloud
				.callFunction({
					name: 'images',
					data: { type: 'mpweixin', categoryId: id }
				})
				.then(res => {
					this.imageList = res.result.data;
					if (this.imageList && this.imageList.length > 0) {
						if (num < 0) {
							this.currentImage = this.imageList[this.imageList.length - 1];
						} else {
							this.currentImage = this.imageList[0];
						}
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
		 * 初始化
		 */
		async init() {
			this.userInfo = uni.getStorageSync('user_info');
		},
		/**
		 * @param {Object} item
		 * 切换分类
		 */
		switchCategory(item, num) {
			let info = this.categoriesList.filter(el => el.selected);
			if (info && info.length > 0) {
				info[0].selected = false;
			}
			this.$set(item, 'selected', true);
			this.getImagesList(item._id, num);
		},
		/**
		 * @param {Object} num
		 * 切换图片
		 */
		switchAvatar(num) {
			let currentIndex = this.imageList.findIndex(el => el._id === this.currentImage._id);
			if ((num > 0 && currentIndex < this.imageList.length - 1) || (num < 0 && currentIndex > 0)) {
				currentIndex += num;
				this.currentImage = this.imageList[currentIndex];
			} else {
				let currentType = this.categoriesList.findIndex(data => data.selected);
				currentType += num;
				if (this.categoriesList[currentType] && this.categoriesList[currentType]._id) {
					this.switchCategory(this.categoriesList[currentType], num);
				}
			}
		},
		/**
		 * @param {Object} e
		 * 设置挂件位置
		 */
		touchStart(e) {
			if (e.target.id == 'mask-img') {
				this.touch_target = 'mask-img';
				this.showBorder = true;
			} else if (e.target.id == 'drag-img') {
				this.touch_target = 'drag-img';
			} else {
				this.touch_target = '';
			}

			if (this.touch_target != '') {
				this.start_x = e.touches[0].clientX;
				this.start_y = e.touches[0].clientY;
			}
		},
		touchMove(e) {
			let current_x = e.touches[0].clientX;
			let current_y = e.touches[0].clientY;
			let moved_x = current_x - this.start_x;
			let moved_y = current_y - this.start_y;
			if (this.touch_target == 'mask-img') {
				this.maskCenterX = this.maskCenterX + moved_x;
				this.maskCenterY = this.maskCenterY + moved_y;
				this.handleCenterX = this.handleCenterX + moved_x;
				this.handleCenterY = this.handleCenterY + moved_y;
			}
			if (this.touch_target == 'drag-img') {
				this.handleCenterX = this.handleCenterX + moved_x; // 190
				this.handleCenterY = this.handleCenterY + moved_y; // 200
				let diff_x_before = this.handle_center_x - this.mask_center_x; // 190
				let diff_y_before = this.handle_center_y - this.mask_center_y; // 190
				let diff_x_after = this.handleCenterX - this.mask_center_x; // 190
				let diff_y_after = this.handleCenterY - this.mask_center_y; // 200
				let distance_before = Math.sqrt(diff_x_before * diff_x_before + diff_y_before * diff_y_before);
				let distance_after = Math.sqrt(diff_x_after * diff_x_after + diff_y_after * diff_y_after);
				let angle_before = (Math.atan2(diff_y_before, diff_x_before) / Math.PI) * 180;
				let angle_after = (Math.atan2(diff_y_after, diff_x_after) / Math.PI) * 180;
				this.scale = (distance_after / distance_before) * this.scaleCurrent;
				this.rotate = angle_after - angle_before + this.rotateCurrent;
			}
			this.start_x = current_x;
			this.start_y = current_y;
		},
		touchEnd(e) {
			this.mask_center_x = this.maskCenterX;
			this.mask_center_y = this.maskCenterY;
			this.handle_center_x = this.handleCenterX;
			this.handle_center_y = this.handleCenterY;
			this.touch_target = '';
			this.scaleCurrent = this.scale;
			this.rotateCurrent = this.rotate;
		},
		/**
		 * 不显示border
		 */
		touchAvatarBg() {
			this.showBorder = false;
		},
		/**
		 * @param {Object} item
		 * 图片点击事件
		 */
		imageClick(item) {
			this.currentImage = item;
		},
		/**
		 * @param {Object} val
		 * 是否展示左右切换
		 */
		showSwitch(val) {
			let currentType = this.categoriesList.findIndex(data => data.selected);
			let currentIndex = this.imageList.findIndex(el => el._id === this.currentImage._id);
			let res = (val < 0 && currentType <= 0 && currentIndex <= 0) || (val > 0 && currentType >= this.categoriesList.length - 1 && currentIndex >= this.imageList.length - 1);
			return !res;
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
		 * 生成新头像
		 */
		async shareFc() {
			if (!this.avatarImage) {
				uni.showToast({
					title: '请先获取头像',
					icon: 'none'
				});
				return;
			}
			try {
				uni.showLoading({
					title: '加载中',
					mask: true
				});
				let mask_center_x = this.mask_center_x;
				let mask_center_y = this.mask_center_y;
				let that = this;
				var query = wx.createSelectorQuery();
				query.select('#avatar-img').boundingClientRect();
				query.exec(res => {
					mask_center_x = mask_center_x - res[0].left;
					mask_center_y = mask_center_y - res[0].top;
					const context = wx.createCanvasContext(that.canvasId);
					const mask_size = 100 * that.scale;

					context.clearRect(0, 0, that.cansWidth, that.cansHeight);
					console.log(that.avatarImage);
					context.drawImage(that.avatarImage, 0, 0, that.cansWidth, that.cansHeight);
					context.translate(mask_center_x, mask_center_y);
					context.rotate((that.rotate * Math.PI) / 180);
					context.drawImage(that.currentImage.image_url, -mask_size / 2, -mask_size / 2, mask_size, mask_size);
					
					context.draw(false, () => {
						that.saveCans();
					});
				});
			} catch (e) {
				uni.hideLoading();
				uni.showToast(JSON.stringify(e));
			}
		},
		saveCans() {
			uni.canvasToTempFilePath(
				{
					x: 0,
					y: 0,
					height: this.cansWidth,
					width: this.cansHeight,
					destWidth: this.cansWidth * 3,
					destHeight: this.cansHeight * 3,
					canvasId: this.canvasId,
					success: res => {
						uni.hideLoading();
						this.posterImage =res.tempFilePath 
						// this.savefile();
					},
					fail(res) {
						uni.hideLoading();
					}
				},
				this
			);
		},
		/**
		 * 保存图片
		 */
		savefile() {
			//获取相册授权
			let _self = this;
			uni.getSetting({
				success(res) {
					if (!res.authSetting['scope.writePhotosAlbum']) {
						uni.authorize({
							scope: 'scope.writePhotosAlbum',
							success() {
								//这里是用户同意授权后的回调
								_self.saveImgToLocal();
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
						_self.saveImgToLocal();
					}
				}
			});
		},
		/**
		 * @param {Object} e
		 * 保存到相册
		 */
		saveImgToLocal(e) {
			let _self = this;
			uni.saveImageToPhotosAlbum({
				filePath: _self.posterImage,
				success: function() {
					_self.saveImageInfo();
					uni.setStorageSync('currentImage', _self.posterImage);
				},
				fail: (e)=> {
					uni.hideLoading();
					uni.showToast({
						title: '保存失败',
						icon: 'none'
					});
				}
			});
		},
		/**
		 * 头像
		 */
		saveImageInfo() {
			uniCloud
				.callFunction({
					name: 'images',
					data: { imageInfo: this.currentImage, type: 'imageUsed' }
				})
				.then(res => {
					uni.hideLoading();
					uni.navigateTo({
						url: '/pages/save-success/save-success'
					});
				});
		},
		/**
		 * @param {Object} type
		 * 获取头像
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
					let info = data.userInfo.avatarUrl;
					if (type === 'createImages') {
						_this.avatarImage = info.substring(0, info.lastIndexOf('/') + 1) + '0';
						uni.setStorageSync('avatar_image', _this.avatarImage);
					}
					_this.postUserInfo(result.userInfo.nickName, type);
				},
				fail(fall) {}
			});
		},
		/**
		 * @param {Object} nickName
		 * 存储用户数据
		 */
		async postUserInfo(nickName, type) {
			let that = this;
			this.code = await this.getWeixinCode();
			if (type === 'userLogin') {
				uni.showLoading({
					title: '加载中',
					mask: true
				});
			}
			uniCloud
				.callFunction({
					name: 'user_mpweixin',
					data: { code: that.code, avatarImage: that.avatarImage, nickName, type }
				})
				.then(res => {
					uni.setStorageSync('user_info', res.result);
					this.userInfo = res.result;
					if (type === 'userLogin') {
						uni.hideLoading();
						this.navOriginal();
					}
				});
		},
		navOriginal() {
			uni.navigateTo({
				url: '/pages/original-avatar/original-avatar'
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	background-size: 100% 100%;
	padding-top: 200rpx;
	.all-back {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		min-height: 100vh;
		width: 750rpx;
	}
	.top-content {
		width: 610rpx;
		background-color: #ffffff;
		margin: 30rpx;
		border-radius: 50rpx;
		padding: 0 40rpx 30rpx;
		position: relative;
		.top-title {
			display: flex;
			align-items: center;
			.title-unit {
				padding: 40rpx 20rpx;
				font-size: 30rpx;
			}
			.title-select {
				font-size: 30rpx;
				font-weight: bold;
				color: #ff4500;
			}
		}
		.image-div {
			display: flex;
			align-items: center;
			padding-left: 20rpx;
			padding-bottom: 20rpx;
			background-color: #ffffff;
			image {
				width: 120rpx;
				height: 120rpx;
				border: 1rpx solid #f8f8f8;
				box-shadow: 0px -5px 15px 0px rgba(224, 224, 224, 0.4);
				flex-shrink: 0;
			}
			.image-margin {
				margin: 0 20rpx;
			}
		}
	}
	.image-card {
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
		.iconfont {
			color: #f7f8fa;
			font-size: 80rpx;
			font-weight: bold;
		}
	}
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}

.avatar-div {
	height: 380rpx;
	margin-right: 40rpx;
	position: relative;
	width: 380rpx;
	.img {
		background-color: #fff;
		border-radius: 48rpx;
		height: 360rpx;
		position: absolute;
		width: 360rpx;
		z-index: 0;
	}
	.avatar-default {
		border-radius: 48rpx;
		height: 100%;
		left: 0;
		position: absolute;
		top: 0;
		width: 100%;
		z-index: 10;
	}
	.avatar-border {
		box-sizing: border-box;
		border: 6rpx dashed #ffffff;
	}
	.drag-div {
		position: absolute;
		box-sizing: border-box;
		top: 360rpx;
		right: 360rpx;
		width: 60rpx;
		height: 60rpx;
		background-color: #ffffff;
		border-radius: 50%;
		padding: 10rpx;
		z-index: 9999999;
		display: flex;
		justify-content: center;
		align-items: center;
		.drag-img {
			width: 100%;
			height: 100%;
		}
	}
}

.avatar-div,
.empty-view {
	-webkit-box-orient: vertical;
	-webkit-box-direction: normal;
	-webkit-box-pack: center;
	-webkit-box-align: center;
	align-items: center;
	display: flex;
	flex-direction: column;
	justify-content: center;
	z-index: 1;
}

.empty {
	height: 100px;
	margin-bottom: 24px;
	width: 100px;
}

.container {
	background-color: #fbebe1;
	min-height: 100vh;
	overflow: hidden;
}
.photo-main-view {
	display: flex;
	justify-content: space-between;
	width: 690rpx;
	margin: 30rpx 30rpx 0;
}
.icon-div {
	position: relative;
	height: 80rpx;
	.icon-zuo {
		position: absolute;
		left: 0;
	}
	.icon-you {
		position: absolute;
		right: 0;
	}
}
.action-btn {
	background: #fff;
	border: 1rpx solid #efefef;
	border-radius: 48rpx;
	box-shadow: 0 12rpx 16rpx -8rpx rgba(0, 0, 0, 0.1);
	color: #4d4d4d;
	font-weight: bolder;
	height: 90rpx;
	line-height: 90rpx;
	font-size: 30rpx;
	padding: 0 60rpx;
}
.btn-margin {
	margin-bottom: 50rpx;
}
.btn-primary {
	background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
	border: 1rpx solid #ff7852;
	border-radius: 48rpx;
	box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
	color: #fff;
}
.btn-card {
	padding: 80rpx 30rpx 30rpx;
	box-sizing: border-box;
	width: 750rpx;
	.primary-btn {
		width: 330rpx;
		display: inline-block;
		margin-right: 30rpx;
		background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
		border: 1rpx solid #ff7852;
		border-radius: 48rpx;
		box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
		color: #fff;
		padding: 0;
		font-size: 30rpx;
		height: 90rpx;
		line-height: 90rpx;
	}
	.share-btn {
		width: 330rpx;
		display: inline-block;
		background: linear-gradient(97.71deg, #ffd01e, #ff8917 60%);
		border: 1rpx solid #ff7852;
		border-radius: 48rpx;
		box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
		color: #fff;
		padding: 0;
		font-size: 30rpx;
		height: 90rpx;
		line-height: 90rpx;
	}
}

.hideCanvasView {
	position: relative;
}

.hideCanvas {
	position: fixed;
	top: -99999upx;
	left: -99999upx;
	z-index: -99999;
}
</style>
