<template>
	<view class="content">
		<!-- #ifdef MP-WEIXIN -->
		<button type="default" @tap="loginByWeixin">微信登录</button>
		<button type="default" @tap="code2SessionWeixin">获取微信openid</button>
		<button type="default" @tap="bindWeixin">绑定微信</button>
		<button type="default" @tap="unbindWeixin">解绑微信</button>
		<!-- #endif -->
		<!-- #ifdef APP-PLUS -->
		<template v-if="hasWeixinAuth">
			<button type="default" @tap="loginByWeixin">微信登录</button>
			<button type="default" @tap="bindWeixin">绑定微信</button>
			<button type="default" @tap="unbindWeixin">解绑微信</button>
		</template>
		<template v-else>
			<view class="tips">未包含微信登录模块</view>
		</template>
		<!-- #endif -->
	</view>
</template>

<script>
	let weixinAuthService
	export default {
		data() {
			return {
				hasWeixinAuth: false
			}
		},
		onLoad() {
			// #ifdef APP-PLUS
			plus.oauth.getServices((services) => {
				weixinAuthService = services.find((service) => {
					return service.id === 'weixin'
				})
				if (weixinAuthService) {
					this.hasWeixinAuth = true
				}
			});
			// #endif	
		},
		methods: {
			getWeixinCode() {
				return new Promise((resolve, reject) => {
					uni.login({
						provider: 'weixin',
						success(res) {
							console.log('----------------')
							console.log(res);
							resolve(res.code)
						},
						fail(err) {
							console.log('----------------')
							console.log(err);
							reject(new Error('微信登录失败'))
						}
					})
				})
			},
			code2SessionWeixin() {
				this.getWeixinCode().then((code) => {
					return uniCloud.callFunction({
						name: 'user-center',
						data: {
							action: 'code2SessionWeixin',
							params: {
								code,
							}
						}
					})
				}).then((res) => {
					uni.showModal({
						showCancel: false,
						content: JSON.stringify(res.result)
					})
				}).catch((e) => {
					console.error(e)
					uni.showModal({
						showCancel: false,
						content: '微信登录失败，请稍后再试'
					})
				})
			},
			unbindWeixin() {
				uniCloud.callFunction({
					name: 'user-center',
					data: {
						action: 'unbindWeixin'
					},
					success(res) {
						uni.showModal({
							showCancel: false,
							content: JSON.stringify(res.result)
						})
					},
					fail(e) {
						console.error(e)
						uni.showModal({
							showCancel: false,
							content: '微信解绑失败，请稍后再试'
						})
					}
				})
			},
			bindWeixin() {
				this.getWeixinCode().then((code) => {
					return uniCloud.callFunction({
						name: 'user-center',
						data: {
							action: 'bindWeixin',
							params: {
								code: code,
							}
						}
					})
				}).then((res) => {
					uni.showModal({
						showCancel: false,
						content: JSON.stringify(res.result)
					})
				}).catch(() => {
					uni.showModal({
						showCancel: false,
						content: '微信绑定失败，请稍后再试'
					})
				})
			},
			loginByWeixin() {
				this.getWeixinCode().then((code) => {
					return uniCloud.callFunction({
						name: 'user-center',
						data: {
							action: 'loginByWeixin',
							params: {
								code,
							}
						}
					})
				}).then((res) => {
					uni.showModal({
						showCancel: false,
						content: JSON.stringify(res.result)
					})
					if (res.result.code === 0) {
						uni.setStorageSync('uni_id_token', res.result.token)
						uni.setStorageSync('uni_id_token_expired', res.result.tokenExpired)
					}
				}).catch((e) => {
					console.error(e)
					uni.showModal({
						showCancel: false,
						content: '微信登录失败，请稍后再试'
					})
				})
			},
		}
	}
</script>

<style>

</style>
