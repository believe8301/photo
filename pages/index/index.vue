<template>
	<view class="content" :style="{'background-image': 'url('+(indexBg.imageUrl||'')+')'}" @click.native="touchAvatarBg(false)">
		<view class="hideCanvas"><canvas class="default_PosterCanvasId" canvas-id="default_PosterCanvasId"></canvas></view>

		<view class="image-card">
			<view class="switch-div"><view class="icon-xiangzuo iconfont" v-if="showSwitch(-1)" @click.stop="switchAvatar(-1)"></view></view>
			<view class="avatar-div " id="avatar-container" @click.stop @touchstart="touchStart" @touchend="touchEnd" @touchmove="touchMove">
				<image class="img" id="avatar-img" :src="avatarImage || defaultImage" @click="touchAvatarBg(false)"></image>
				<image
					class="avatar-default"
					:class="{ 'avatar-border': showBorder && currentImage.drag_state }"
					:style="{
						top: maskCenterY - maskSize / 2 - 2 + 'px',
						left: maskCenterX - maskSize / 2 - 2 + 'px',
						transform: 'rotate(' + rotate + 'deg)' + 'scale(' + scale + ')'
					}"
					id="mask-img"
					:src="currentImage.image_url"
					v-if="currentImage && currentImage.image_url"
					@click="touchAvatarBg(true)"
				></image>
				<view
					class="drag-div"
					:style="{ top: handleCenterY - 10 + 'px', left: handleCenterX - 10 + 'px', width: handleWidth + 'rpx', height: handleWidth + 'rpx' }"
					v-if="showBorder && currentImage && currentImage.drag_state"
				>
					<image class="drag-img" id="drag-img" src="/static/images/drag.svg"></image>
				</view>
			</view>
			<view class="switch-div"><view class="icon-xiangzuo iconfont rotate-level" v-if="showSwitch(1)" @click.stop="switchAvatar(1)"></view></view>
		</view>
		<view class="btn-card">
			<view v-if="userInfo" class="btn-right">
				<button class="primary-btn action-btn" @click.stop="getUserProfile('createImages')">获取头像</button>
				<button class="primary-btn" @click.stop="chooseImages('selectedImage')">选择图片</button>
			</view>
			<view v-else class="btn-right">
				<button class="primary-btn action-btn" open-type="getUserInfo" @click.stop="getUserProfile('createImages')">获取头像</button>
				<button class="primary-btn" open-type="getUserInfo" @click.stop="getUserProfile('selectedImage')">选择图片</button>
			</view>
			<view class="btn-right">
				<button open-type="share" class="primary-btn share-btn" @click.stop>分享给好友</button>
				<view class="save-btn btn-right" @click.stop="shareFc()">
					<text class="iconfont icon-wancheng"></text>
					保存
				</view>
			</view>
		</view>
		<view class="top-content" @click.stop>
			<scroll-view scroll-x :show-scrollbar="false">
				<view class="top-title">
					<view class="title-unit" :class="{ 'title-select': item.selected }" v-for="(item, index) in categoriesList" :key="item._id" @click="switchCategory(item)">
						{{ item.name }}
					</view>
				</view>
			</scroll-view>
			<scroll-view scroll-x :show-scrollbar="false" class="scroll-view">
				<view class="image-div">
					<view :class="{ 'image-margin': index !== 0 }" v-for="(info, index) in imageList" :key="info._id" @click="imageClick(info)">
						<image :src="info.image_url"></image>
					</view>
				</view>
			</scroll-view>
		</view>
		<view v-if="userInfo" class="history-card">
			<button class="history-btn" @click.stop="navOriginal()">
				查看原头像
				<view class="icon-div"><text class="iconfont icon-xiangzuo"></text></view>
			</button>
		</view>
		<view v-else class="history-card">
			<button class="history-btn" open-type="getUserInfo" @click.stop="getUserProfile('userLogin')">
				查看原头像
				<view class="icon-div"><text class="iconfont icon-xiangzuo"></text></view>
			</button>
		</view>
		<view class="ad-div" v-if="adState && adInfo && adInfo.imageUrl">
			<view class="icon-guanbi iconfont" @click.stop="adState = false"></view>
			<view class="ad-card" v-if="adInfo && adInfo.imageUrl" :style="{ 'background-image': 'url(' + adInfo.imageUrl + ')' }" @click="navSortition()"></view>
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
			indexBg:  {},
			defaultImage: 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/c33782ca-cd2f-4bfc-84eb-0713c52f522f.svg',
			currentImage: {},
			currentIndex: 0,
			imageList: [],
			categoriesList: [],
			shareInfo: {},
			adInfo: {},
			adState: true,
			showBorder: true, // 默认是否显示边框
			maskCenterX: uni.upx2px(590) / 2,
			maskCenterY: uni.upx2px(590) / 2,
			handleCenterX: uni.upx2px(560), // 360
			handleCenterY: uni.upx2px(570), // 360
			handleWidth: 50, // 360
			maskSize: uni.upx2px(590),
			scale: 1,
			rotate: 0,
			mask_center_x: uni.upx2px(590) / 2,
			mask_center_y: uni.upx2px(590) / 2,
			handle_center_x: uni.upx2px(560), // 360
			handle_center_y: uni.upx2px(570), // 360
			scaleCurrent: 1,
			rotateCurrent: 0,
			touch_target: '',
			start_x: 0,
			start_y: 0,
			startInfo: 0,
			cansBorder: uni.upx2px(590), // 宽度 px
			bgIndex: 0
		};
	},
	onLoad() {
		this.init();
		this.initBg();
		this.getCategoriesList();
	},
	onShow() {
		this.getShareInfo();
	},
	onReady() {
		var query = wx.createSelectorQuery();
		query.select('.avatar-div').boundingClientRect();
		query.exec(res => {
			this.startInfo = res[0];
		});
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
						this.shareInfo = res.result.data.find(el => el.code === 'mpwx_share');
						this.adInfo = res.result.data.find(el => el.code === 'index_ad');
						this.indexBg = res.result.data.find(el => el.code === 'index_bg');
						uni.setStorageSync('shareInfo', this.shareInfo);
						uni.setStorageSync('background_info', res.result.data);
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
						if (!(this.currentImage && this.currentImage.drag_state)) {
							this.initImage();
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
			this.code = await this.getWeixinCode();
			this.userInfo = uni.getStorageSync('user_info');
		},
		/**
		 * 初始化
		 */
		initBg() {
			if (uni.getStorageSync('background_info') && uni.getStorageSync('background_info').length) {
				let info = uni.getStorageSync('background_info')
				this.shareInfo = info.find(el => el.code === 'mpwx_share');
				this.adInfo = info.find(el => el.code === 'index_ad');
				this.indexBg = info.find(el => el.code === 'index_bg');
			} else if (this.bgIndex < 5) {
				this.bgIndex++
				setTimeout(()=> {
					this.initBg()
				},300)
			} else {
				this.getShareInfo()
			}
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
			this.touchAvatarBg(true);
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
			this.$nextTick(() => {
				if (!(this.currentImage && this.currentImage.drag_state)) {
					this.initImage();
				}
			});
		},
		/**
		 * @param {Object} e
		 * 设置挂件位置
		 */
		touchStart(e) {
			if (!(this.currentImage && this.currentImage.drag_state)) {
				return;
			}
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
			if (!(this.currentImage && this.currentImage.drag_state)) {
				return;
			}
			let current_x = e.touches[0].clientX;
			let current_y = e.touches[0].clientY;
			let moved_x = current_x - this.start_x;
			let moved_y = current_y - this.start_y;
			if (this.touch_target === 'mask-img') {
				this.maskCenterX = this.maskCenterX + moved_x;
				this.maskCenterY = this.maskCenterY + moved_y;
				this.handleCenterX = this.handleCenterX + moved_x;
				this.handleCenterY = this.handleCenterY + moved_y;
			}
			if (this.touch_target === 'drag-img') {
				this.handleCenterX = this.handleCenterX + moved_x;
				this.handleCenterY = this.handleCenterY + moved_y;
				let diff_x_before = this.handle_center_x - this.mask_center_x;
				let diff_y_before = this.handle_center_y - this.mask_center_y;
				let diff_x_after = this.handleCenterX - this.mask_center_x;
				let diff_y_after = this.handleCenterY - this.mask_center_y;
				let distance_before = Math.sqrt(diff_x_before * diff_x_before + diff_y_before * diff_y_before);
				let distance_after = Math.sqrt(diff_x_after * diff_x_after + diff_y_after * diff_y_after);
				let angle_before = (Math.atan2(diff_y_before, diff_x_before) / Math.PI) * 180;
				let angle_after = (Math.atan2(diff_y_after, diff_x_after) / Math.PI) * 180;
				this.scale = (distance_after / distance_before) * this.scaleCurrent;
				this.rotate = angle_after - angle_before + this.rotateCurrent;
				let width = this.scale * 100;
				if (width > 50) {
					this.handleWidth = 50;
				} else if (width < 10) {
					this.handleWidth = 10;
				} else {
					this.handleWidth = width;
				}
			}
			this.start_x = current_x;
			this.start_y = current_y;
		},
		touchEnd(e) {
			if (!(this.currentImage && this.currentImage.drag_state)) {
				return;
			}
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
		touchAvatarBg(state) {
			this.showBorder = state;
		},
		/**
		 * @param {Object} item
		 * 图片点击事件
		 */
		imageClick(item) {
			this.currentImage = item;
			if (!(this.currentImage && this.currentImage.drag_state)) {
				this.initImage();
			}
		},
		/**
		 * 还原设置
		 */
		initImage() {
			this.showBorder = true; // 默认是否显示边框
			this.maskCenterX = uni.upx2px(590) / 2;
			this.maskCenterY = uni.upx2px(590) / 2;
			this.handleCenterX = uni.upx2px(560);
			this.handleCenterY = uni.upx2px(570);
			this.maskSize = uni.upx2px(590);
			this.scale = 1;
			this.rotate = 0;
			this.mask_center_x = uni.upx2px(590) / 2;
			this.mask_center_y = uni.upx2px(590) / 2;
			this.handle_center_x = uni.upx2px(560);
			this.handle_center_y = uni.upx2px(570);
			this.scaleCurrent = 1;
			this.rotateCurrent = 0;
			this.touch_target = '';
			this.start_x = 0;
			this.start_y = 0;
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
			uni.showLoading({
				title: '加载中',
				mask: true
			});
			const context = uni.createCanvasContext('default_PosterCanvasId', this);
			const mask_size=this.scale*100
			context.clearRect(0, 0, this.cansBorder, this.cansBorder);
			uni.getImageInfo({
				src: this.avatarImage,
				success: response1 => {
					context.drawImage(response1.path, 0, 0, this.cansBorder, this.cansBorder);
					uni.getImageInfo({
						src: this.currentImage.image_url,
						success: response2 => {
							context.translate(this.mask_center_x, this.mask_center_y);
							context.rotate((this.rotate * Math.PI) / 180);
							context.drawImage(response2.path, -mask_size / 2, -mask_size / 2, mask_size, mask_size);
							context.draw(false, () => {
								this.saveCans();
							});
						}
					});
				}
			});
		},
		saveCans() {
			uni.canvasToTempFilePath(
				{
					x: 0,
					y: 0,
					height: this.cansBorder,
					width: this.cansBorder,
					destWidth: this.cansBorder,
					destHeight: this.cansBorder,
					canvasId: this.canvasId,
					success: res => {
						this.posterImage = res.tempFilePath;
						this.savefile();
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
				fail: e => {
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
			uniCloud.callFunction({
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
			if (type === 'userLogin' || type === 'selectedImage') {
				uni.showLoading({
					title: '加载中',
					mask: true
				});
			}
			uniCloud.callFunction({
				name: 'user_mpweixin',
				data: { code: that.code, avatarImage: that.avatarImage, nickName, type: type === 'selectedImage'? 'userLogin': type }
			}).then(res => {
					uni.setStorageSync('user_info', res.result);
					this.userInfo = res.result;
					if (type === 'userLogin') {
						uni.hideLoading();
						this.navOriginal();
					} else if (type === 'selectedImage') {
						this.chooseImages(type);
					}
			});
		},
		/**
		 * 选择图片
		 */
		chooseImages(type) {
			uni.hideLoading()
			uni.chooseImage({
				count: 1, //默认9
				sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
				sourceType: ['album', 'camera'], //从相册选择
				success: res => {
					let filePath = res.tempFilePaths[0];
					uniCloud.uploadFile({
						filePath: filePath,
						cloudPath: `userChooseImage-${new Date().getTime()}.png`
					}).then(res => {
						this.avatarImage = res['fileID'];
						//获取到上传到云储存的url地址
						uniCloud.callFunction({
							name: 'user_mpweixin',
							data: { userId: this.userInfo._id, avatarImage: this.avatarImage, type }
						}).then();
					});
				}
			});
		},
		navOriginal() {
			uni.navigateTo({
				url: '/pages/original-avatar/original-avatar'
			});
		},
		navSortition() {
			uni.navigateTo({
				url: '/pages/sortition/sortition'
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	background-size: 100% 100%;
	padding: 200rpx 0 0;
	min-height: 100vh;
	box-sizing: border-box;
	background-image: url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/3da3f4b5-271b-48f5-b13f-acb0d362ed91.jpg');
	.top-content {
		width: 610rpx;
		background-color: #ffffff;
		margin: 0 30rpx;
		border-radius: 50rpx;
		padding: 0 40rpx 30rpx;
		position: relative;
		.top-title {
			display: flex;
			align-items: center;
			flex-wrap: nowrap;
			.title-unit {
				padding: 40rpx 20rpx;
				white-space: nowrap;
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
		.switch-div {
			width: 80rpx;
			height: 300rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			overflow: hidden;
			.iconfont {
				font-size: 78rpx;
				font-weight: bold;
				background-image: -webkit-linear-gradient(180deg, #f7f8fa, #ff4d42 88.36%);
				-webkit-text-fill-color: transparent;
				-webkit-background-clip: text;
			}
			.rotate-level {
				transform: rotateY(180deg);
			}
		}
		.avatar-div {
			height: 590rpx;
			position: relative;
			width: 590rpx;
			-webkit-box-orient: vertical;
			-webkit-box-direction: normal;
			-webkit-box-pack: center;
			-webkit-box-align: center;
			align-items: center;
			display: flex;
			flex-direction: column;
			justify-content: center;
			z-index: 1;
			.img {
				background-color: #fff;
				border-radius: 48rpx;
				height: 100%;
				position: absolute;
				width: 100%;
				z-index: 0;
			}
			.avatar-default {
				box-sizing: content-box;
				border-radius: 48rpx;
				height: 100%;
				width: 100%;
				margin: 6rpx;
				position: absolute;
				top: 0;
				left: 0;
				z-index: 10;
			}
			.avatar-border {
				margin: 0;
				border: 6rpx dashed #ffffff;
			}
			.drag-div {
				position: absolute;
				top: 590rpx;
				right: 590rpx;
				width: 50rpx;
				height: 50rpx;
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
	}
	.btn-card {
		padding: 40rpx 30rpx 30rpx;
		box-sizing: border-box;
		width: 750rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		.primary-btn {
			display: inline-block;
			margin-right: 20rpx;
			background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
			border: 1rpx solid #ff7852;
			border-radius: 16rpx;
			box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
			color: #fff;
			padding: 0 20rpx;
			font-size: 28rpx;
			height: 70rpx;
			line-height: 68rpx;
		}
		.action-btn {
			background: #fff;
			border: 1rpx solid #efefef;
			box-shadow: 0 12rpx 16rpx -8rpx rgba(0, 0, 0, 0.1);
			color: #4d4d4d;
		}
		.share-btn {
			background: linear-gradient(97.71deg, #ffd01e, #ff8917 60%);
		}
		.btn-right {
			display: flex;
			align-items: center;
		}
		.save-btn {
			color: #ff4d42;
			font-size: 30rpx;
			font-weight: bold;
		}
	}
	.ad-div {
		position: fixed;
		top: 300rpx;
		right: 0;
		box-sizing: border-box;
		z-index: 99999;
		display: flex;
		align-items: flex-end;
		flex-direction: column;
		.icon-guanbi {
			font-size: 36rpx;
			color: #ffffff;
		}
		.ad-card {
			background-size: cover;
			width: 200rpx;
			height: 200rpx;
			animation: d-light 2s linear infinite;
			font-size: 20rpx;
			color: #ffffff;
		}
	}
	.history-card {
		display: flex;
		justify-content: flex-end;
		align-items: flex-end;
		padding-top: 20rpx;
		.history-btn {
			background-color: transparent;
			border-width: 0;
			font-size: 30rpx;
			font-weight: bold;
			color: #ffffff;
			display: inline-block;
			margin-left: 0;
			margin-right: 0;
			background: linear-gradient(97.71deg, #ffd01e, #ff8917 60%);
			height: 70rpx;
			line-height: 70rpx;
			border-radius: 40rpx 0 0 40rpx;
			padding-right: 5rpx;
			padding-left: 30rpx;
			.icon-div {
				display: inline-block;
				transform: rotateY(180deg);
				.icon-xiangzuo {
					font-size: 30rpx;
				}
			}
		}
		.history-btn::after {
			border-left-width: 0;
			border-right-width: 0;
			border-top-width: 0;
			border-bottom-width: 0;
		}
	}

	.hideCanvas {
		position: fixed;
		top: -99999upx;
		left: -99999upx;
		z-index: -99999;
		.default_PosterCanvasId {
			width: 590rpx;
			height: 590rpx;
		}
	}
}
@keyframes d-light {
	0% {
		transform: rotate(0deg);
	}
	25% {
		transform: rotate(10deg);
	}
	75% {
		transform: rotate(-10deg);
	}
	10% {
		transform: rotate(0deg);
	}
}
</style>
