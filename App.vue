<script>
	export default {
		data() {
			return {
				shareInfo: uni.getStorageSync('shareInfo')
			}
		},
		onLaunch: function() {
			uni.removeStorageSync('avatar_image');
			uni.removeStorageSync('user_info');
		},
		onShow: function() {
			this.getShareInfo()
		},
		onHide: function() {
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
		}
	}
</script>

<style>
	/*每个页面公共css*/
	@import url("./static/icon/icon.css");
</style>
