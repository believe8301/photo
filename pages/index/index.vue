<template>
	<view class="content">
		<image src="../../static/background.jpg" class="all-back"></image>
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
				<!--  -->
				<view class="avatar-div " id="avatar-container">
					<image class="img" id="avatar-img" :src="avatarImage"></image>
					<view class="empty-view " v-if="!avatarImage"><image class="empty" src="../../static/images/avatar_empty.svg"></image></view>
					<image class="avatar-default " :src="currentImage.image_url" v-if="currentImage && currentImage.image_url"></image>
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
			currentImage: {},
			currentIndex: 0,
			imageList: [],
			categoriesList: [],
			shareInfo: {}
		};
	},
	onLoad() {
		this.init();
		this.getCategoriesList();
		this.getShareInfo();
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
		getImagesList(id) {
			uni.showLoading({
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
						this.currentImage = this.imageList[0];
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
		switchCategory(item) {
			let info = this.categoriesList.filter(el => el.selected);
			if (info && info.length > 0) {
				info[0].selected = false;
			}
			this.$set(item, 'selected', true);
			this.getImagesList(item._id);
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
				if (currentType >= 0 && currentType < this.imageList.length) {
					this.switchCategory(this.categoriesList[currentType]);
				}
			}
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
			let res =
				(val < 0 && currentType <= 0 && currentIndex <= 0) || (val > 0 && currentType >= this.categoriesList.length - 1 && currentIndex >= this.categoriesList.length - 1);
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
				_app.log('准备生成:' + new Date());
				const d = await getSharePoster({
					_this: this, //若在组件中使用 必传
					type: 'testShareType',
					formData: {
						//访问接口获取背景图携带自定义数据
					},
					backgroundImage: this.avatarImage,
					posterCanvasId: this.canvasId, //canvasId
					delayTimeScale: 20, //延时系数
					drawArray: ({ bgObj, type, bgScale }) => {
						//可直接return数组，也可以return一个promise对象, 但最终resolve一个数组, 这样就可以方便实现后台可控绘制海报
						return new Promise((rs, rj) => {
							rs([
								{
									type: 'image',
									url: this.currentImage.image_url,
									dx: 0,
									dy: 0,
									infoCallBack(imageInfo) {
										return {
											dWidth: bgObj.width, // 因为设置了圆形图片 所以要乘以2
											dHeight: bgObj.height
										};
									}
								}
							]);
						});
					},
					setCanvasWH: ({ bgObj, type, bgScale }) => {
						// 为动态设置画布宽高的方法，
						this.poster = bgObj;
					}
				});
				_app.log('海报生成成功, 时间:' + new Date() + '， 临时路径: ' + d.poster.tempFilePath);
				this.posterImage = d.poster.tempFilePath;
				this.savefile();
			} catch (e) {
				uni.hideLoading();
				_app.hideLoading();
				_app.showToast(JSON.stringify(e));
			}
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
				fail: function() {
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
	padding: 30rpx 30rpx;
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
