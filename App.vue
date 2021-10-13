<script>
	export default {
		onLaunch: function() {
		},
		onShow: function() {
			uni.removeStorageSync('avatar_image');
			uni.removeStorageSync('user_info');
		},
		onHide: function() {
		}
	}
</script>

<style>
	/*每个页面公共css  */
	@import url("./static/icon/icon.css");
</style>
