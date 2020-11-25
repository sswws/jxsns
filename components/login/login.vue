<template>
	<view class="login" :class="{show:show}">
		<view class="mask" @tap="closeLogin" bind:tap="cancel" catch:touchmove="emptyHandler"></view>
		<view class="container">

			<view class="close-box" @tap="closeLogin" bind:tap="cancel">
				<image class="close-img" src="/static/close.png" />
			</view>

			<view class="p-name" v-if="getUserInfoTag">
				前端交流学习中心欢迎你
			</view>
			<button class="submit-btn" open-type="getUserInfo" @getuserinfo="getWechatUserInfo" v-if="getUserInfoTag">
				<image src="/static/wechat.png" class="wechat-img" />
				<text>一键获取微信信息</text>
			</button>

			<u-form :model="form" ref="uForm" v-show="!getUserInfoTag">
				<u-form-item label="账号" prop="login" label-width="150" required v-if="loginType === 'login'">
					<u-input v-model="form.login" placeholder='输入手机号/邮箱/昵称' />
				</u-form-item>

				<u-form-item label="昵称" prop="name" label-width="150" required v-if="loginType !== 'login'">
					<u-input v-model="form.name" placeholder='2-10个字符' />
				</u-form-item>

				<u-form-item label="电话" prop="phone" label-width="150" required v-if="loginType === 'phone'">
					<u-input v-model="form.phone" placeholder='输入11位手机号码' />
				</u-form-item>
				<u-form-item label="邮箱" prop="email" label-width="150" required v-if="loginType === 'email'">
					<u-input v-model="form.email" placeholder='输入邮箱' />
				</u-form-item>

				<u-form-item label="验证码" prop="code" required label-width="150" v-if="loginType !== 'login'">
					<u-input placeholder="请输入4位验证码" v-model="form.code" type="text"></u-input>
					<u-button slot="right" type="success" size="mini" @click="getCode">获取验证码</u-button>
				</u-form-item>

				<u-form-item label="密码" label-width="150" required prop="password">
					<u-input v-model="form.password" type="password" placeholder='限4-20个字符,区分大小写' />
				</u-form-item>
				<u-form-item label="重复密码" required label-width="150" prop="repassword" v-if="loginType !== 'login'">
					<u-input v-model="form.repassword" type="password" placeholder='再次输入密码' />
				</u-form-item>
				<view class="btns">
					<u-button class="ubtn" @click="submit">提交</u-button>
					<u-button class="ubtn" @click="cancel">取消</u-button>
				</view>
				<view class="type">
					<u-subsection active-color="#007cba" font-size="24" height="52" :list="subsectionList" :current="0" @change="sectionChange"></u-subsection>
				</view>
			</u-form>


			<view class="serve-info">点击登录/注册前端论坛，即表示已阅读并同意</view>
			<view class="serve-text">
				<view @tap="gotoWeb('https://uniapp.dcloud.io/component/mp-weixin-plugin')">《隐私政策》</view>
				<view @tap="gotoWeb('https://developers.weixin.qq.com/miniprogram/dev/framework/')">《用户协议》</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState,
		mapActions
	} from 'vuex'

	export default {
		data() {
			return {
				// 登陆组件是否显示
				show: false,
				// 登陆方式
				loginType: 'login',
				// 登陆方式选择器
				subsectionList: [{
						name: '账号登陆'
					},
					{
						name: '手机注册'
					},
					{
						name: '邮箱注册'
					}
				],
				// 是否获取用户信息
				getUserInfoTag: true,
				// 用户信息输入框
				form: {
					login: '',
					avatar: '',
					name: '',
					phone: '',
					email: '',
					code: '',
					password: '',
					repassword: ''
				},
				// 用户输入规则校验
				rules: {
					login: [{
						validator: (rule, value, callback) => {
							if (this.loginType === 'login') {
								return !this.$u.test.isEmpty(value)
							} else {
								return true
							}
						},
						message: '必填 * 输入内容不许为空',
						trigger: ['change', 'blur'],
					}],
					name: [{
							asyncValidator: (rule, value, callback) => {
								this.$u.api.findUser({
									name: value
								}).then(res => {
									// 如果验证不通过，需要在callback()抛出new Error('错误提示信息')
									if (!!value && res.statusCode === 200) {
										callback(new Error('当前用户昵称已存在'));
									} else {
										// 如果校验通过，也要执行callback()回调
										callback();
									}
								})
							},
							trigger: ['blur'],
						},
						{
							validator: (rule, value, callback) => {
								if (this.loginType !== 'login') {
									// 名字长度为 2-10 位
									return this.$u.test.rangeLength(value, [2, 10])
								} else {
									return true
								}
							},
							message: '必填 * 昵称长度不得小于2位大于10位',
							// 触发器可以同时用blur和change
							trigger: ['change', 'blur'],
						}
					],
					phone: [{
							asyncValidator: (rule, value, callback) => {
								this.$u.api.findUser({
									name: value
								}).then(res => {
									// 如果验证不通过，需要在callback()抛出new Error('错误提示信息')
									if (!!value && res.statusCode === 200) {
										callback(new Error('当前电话已注册'));
									} else {
										// 如果校验通过，也要执行callback()回调
										callback();
									}
								})
							},
							trigger: ['blur'],
						},
						{
							validator: (rule, value, callback) => {
								if (this.loginType === 'phone') {
									// 自带判断 电话号码合法性方法
									return this.$u.test.mobile(value)
								} else {
									return true
								}
							},
							message: '必填 * 手机号码输入不正确',
							// 触发器可以同时用blur和change
							trigger: ['change', 'blur'],
						}
					],
					email: [{
							asyncValidator: (rule, value, callback) => {
								this.$u.api.findUser({
									name: value
								}).then(res => {
									// 如果验证不通过，需要在callback()抛出new Error('错误提示信息')
									if (!!value && res.statusCode === 200) {
										callback(new Error('当前邮箱已注册'));
									} else {
										// 如果校验通过，也要执行callback()回调
										callback();
									}
								})
							},
							trigger: ['blur'],
						},
						{
							validator: (rule, value, callback) => {
								if (this.loginType === 'email') {
									// 自带判断 邮箱地址合法性方法
									return this.$u.test.email(value)
								} else {
									return true
								}
							},
							message: '必填 * 邮箱地址输入不正确',
							// 触发器可以同时用blur和change
							trigger: ['change', 'blur'],
						}
					],
					code: [{
						validator: (rule, value, callback) => {
							if (this.loginType !== 'login') {
								// 名字长度为 2-10 位
								return value == '8888'
							} else {
								return true
							}
						},
						message: '必填 * 验证码长度为4位',
						// 触发器可以同时用blur和change
						trigger: ['change', 'blur'],
					}],
					password: [{
						required: true,
						min: 4,
						max: 20,
						message: '必填  * 请输入 4-20 位密码',
						trigger: ['change', 'blur'],
					}],
					repassword: [{
						validator: (rule, value, callback) => {
							if (this.loginType === 'login' || this.form.password === value) {
								return true
							} else {
								return false
							}

						},
						message: '必填 * 两次密码输入不一致',
						// 触发器可以同时用blur和change
						trigger: ['change', 'blur'],
					}],
				}
			}
		},
		computed: {
			...mapState(['loginState', 'userInfo'])
		},
		// 必须要在onReady生命周期设置校验规则，因为onLoad生命周期组件可能尚未创建完毕
		onReady() {
			this.$refs.uForm.setRules(this.rules);
		},
		async created() {
			// 每次这个组件展开，我们就去判断一下当前 token 是否可以获取新的 token 如果可以获取，咱们就关闭登陆状态
			let res = await this.$u.api.getUserMsg()
			// 点赞消息数量
			if (res.statusCode === 200) {
				this.show = false
				return
			} else {
				this.show = true
			}

			wx.getSetting({
				success: res => {
					if (res.authSetting["scope.userInfo"]) {
						uni.getUserInfo({
							success: res => {
								// 如果用户授权了，则做两件事，第一件事
								this.form.login = res.userInfo.nickName
								this.form.name = res.userInfo.nickName
								this.form.avatar = res.userInfo.avatarUrl
								this.getUserInfoTag = false
							},
							fail: () => {
								console.log('用户未授权！')
							}
						})
					}
				}
			})
		},
		methods: {
			...mapActions(['userLoginAction', 'userLogoutAction']),
			// 关闭弹窗
			closeLogin() {
				this.show = false
			},
			// 打开登陆弹窗
			openLogin() {
				this.show = true
			},
			// 跳转到 H5 页面
			gotoWeb(url) {
				wx.navigateTo({
					url: '/pages/webview/webview?url=' + encodeURI(url)
				});
			},
			// 微信授权
			getWechatUserInfo() {
				uni.getUserInfo({
					success: res => {
						// 进行下一步操作
						this.getUserInfoTag = false
						this.form.login = res.userInfo.nickName
						this.form.name = res.userInfo.nickName
						this.form.avatar = res.userInfo.avatarUrl
					},
					fail: () => {
						console.log('用户未授权！')
					}
				})
			},
			// 获取验证码
			getCode() {
				uni.showModal({
					title: '验证码获取成功',
					content: '8888'
				})
			},
			// 取消表单输入
			cancel() {
				this.form = {
					login: '',
					name: '',
					phone: '',
					email: '',
					code: '',
					password: '',
					repassword: ''
				}
			},
			// 提交表单验证
			submit() {
				this.$refs.uForm.validate(async valid => {
					if (!valid) {
						uni.showToast({
							title: '请检查输入',
							icon: 'loading'
						})
						return false
					}
					switch (this.loginType) {
						case "login":
							let resa = await this.$u.api.userLogin({
								login: this.form.login,
								password: this.form.password
							})
							if (resa.statusCode === 200) {
								// 登陆成功
								this.loginAfter(resa.data.access_token)
							} else {
								uni.showModal({
									title: '登陆失败',
									content: resa.data.message
								})
							}
							break;
						case "phone":
							let resb = await this.$u.api.userRegister({
								// 必须，用户名
								name: this.form.name,
								// 必须，验证码发送模式。
								verifiable_type: 'sms',
								// 必须，用户收到的验证码。(教学阶段验证码统一使用 8888 )
								verifiable_code: '8888',
								// 如果 `verifiable_type` 为 `sms` 则必须, 手机号码。
								phone: this.form.phone,
								// 可选，密码，如果不输入密码，允许用户无密码注册。
								password: this.form.password
							})
							console.log(resb)
							if (resb.statusCode === 201) {
								// 登陆成功
								this.loginAfter(resb.data.token)
							} else {
								uni.showModal({
									title: '登陆失败',
									content: resb.data.message
								})
							}
							break;
						case "email":
							let resc = await this.$u.api.userRegister({
								// 必须，用户名
								name: this.form.name,
								// 必须，验证码发送模式。
								verifiable_type: 'mail',
								// 必须，用户收到的验证码。(教学阶段验证码统一使用 8888 )
								verifiable_code: '8888',
								// 如果 `verifiable_type` 为 `mail` 则必须, E-Mail。
								email: this.form.email,
								// 可选，密码，如果不输入密码，允许用户无密码注册。
								password: this.form.password
							})
							console.log(resc)
							if (resc.statusCode === 201) {
								// 登陆成功
								this.loginAfter(resc.data.token)
							} else {
								uni.showModal({
									title: '登陆失败',
									content: resc.data.message
								})
							}
							break;
						default:
							uni.showToast({
								title: '未知用户状态',
								icon: 'loading'
							})
							break;
					}
				});

			},
			// 注册、登陆成功后设置相关逻辑
			async loginAfter(token) {
				this.show = false
				uni.setStorageSync('token', token)

				// 获取未读消息提示
				let res = await this.$u.api.getUserMsg()
				let name = this.form.name
				if (this.loginType === 'login') {
					name = this.form.login
				}
				let loginInfo = {
					name,
					avatar: this.form.avatar,
					liked: res.data.user.liked,
					commented: res.data.user.commented
				}
				this.userLoginAction(loginInfo)
				uni.$emit('meUserLogin')
				uni.$emit('indexUserLogin')
			},
			// 更改 登陆 注册 方式选择
			sectionChange(index) {
				switch (index) {
					case 1:
						this.loginType = "phone";
						break;
					case 2:
						this.loginType = "email";
						break;
					default:
						this.loginType = "login";
						break;
				}
			}

		}
	}
</script>

<style lang="scss" scoped>
	.login {
		position: fixed;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		display: flex;
		align-items: flex-end;
		opacity: 0;
		transition: opacity 300, z-index 0 300;
		pointer-events: none;
		z-index: 999;

		&.show {
			z-index: 9999;
			opacity: 1;
			pointer-events: auto;
		}

		.mask {
			position: fixed;
			width: 100%;
			height: 100%;
			background-color: rgba($color: #000000, $alpha: 0.3);
		}

		.container {
			z-index: 999;
			display: flex;
			flex-direction: column;
			width: 100%;
			padding: 40upx 20upx;
			background-color: #f1f1f1;
			border-radius: 20upx;
			align-items: center;
			position: relative;

			.p-name {
				margin-top: 48upx;
				font-size: 36upx;
				font-weight: normal;
			}

			.close-box {
				position: absolute;
				right: 32upx;
				top: 38upx;
				width: 56.56upx;
				height: 56.56upx;
				padding: 10upx;

				.close-img {
					width: 100%;
					height: 100%;
				}
			}
		}

		.header {
			display: flex;
			flex-direction: column;
			align-items: center;
			font-size: 28upx;

			.logo-wrap {
				width: 144upx;
				height: 144upx;
				overflow: hidden;
				border-radius: 20upx;
			}

			.logo {
				width: 100%;
				height: 100%;
			}
		}

		.info {
			color: #333;
		}

		.submit-btn {
			width: 642upx;
			height: 88upx;
			margin-top: 60upx;
			margin-bottom: 60upx;
			border-radius: 44upx;
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			color: #fff;
			background-color: #0046f5;
			font-size: 36upx;

			.wechat-img {
				width: 44upx;
				height: 34upx;
				margin: 0 18upx;
			}
		}

		.phone-login {
			color: #0046f5;
			font-size: 28upx;
			margin-top: 40upx;
			border: none;
			background-color: #f1f1f1;
		}

		.cancel-btn {
			width: 100%;
			margin-top: 60upx;
			color: #333;
			background-color: #f1f1f1;
		}

		.serve-info {
			font-size: 22upx;
			margin-top: 20upx;
		}

		.serve-text {
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			font-size: 22upx;
			color: #6079b8;
			margin-top: 10upx;
		}
	}

	button::after {
		border: none;
	}

	.btns {
		margin-top: 20upx;
		text-align: center;

		.ubtn {
			display: inline-block;
			margin: 0 20upx;
		}
	}

	.type {
		margin-top: 80upx;
	}
</style>
