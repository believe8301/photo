<script>
	export default {
		onLaunch: function() {
			uni.removeStorageSync('avatar_image')
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
		}
	}
</script>

<style>
	/*每个页面公共css  */
	@import url("./static/icon/icon.css");
</style>
