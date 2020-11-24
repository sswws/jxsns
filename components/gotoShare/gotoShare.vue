<template>
	<view class="goto-share" v-if="loginState">
		<!-- 个人分享 选择器 -->
		<image class="share-img" src="../../static/carm.png" @tap.stop="showActionsheet()" mode="aspectFit"/>
	</view>
</template>

<script>
	import {
		mapState
	} from 'vuex'
	export default {
		data() {
			return {
				// 类型选择器相关参数配置
				itemList: ["照片", "视频"]
			};
		},
		computed:{
			...mapState(['loginState'])
		},
		methods: {
			// 底部个人分享 弹窗相关操作
			async showActionsheet() {
				uni.showActionSheet({
					itemList: this.itemList,
					success: function(res) {
						if (res.tapIndex == 0) {
							uni.navigateTo({
								url: "/subpages/share/share?type=imgs",
							});
						} else if (res.tapIndex == 1) {
							uni.navigateTo({
								url: "/subpages/share/share?type=video",
							});
						}
					}
				});
			}
		}
	};
</script>
<style lang="scss" scoped>
	.goto-share {

		// 点击分享按钮
		.share-img {
			width: 50upx;
			height: 50upx;
			padding: 20upx;
			border-radius: 50%;
			box-shadow: 0 0 5px 0 rgb(66, 66, 66);
			background-color: #3c3f86;
			position: fixed;
			bottom: 20upx;
			// #ifdef H5
			bottom: 120upx;
			// #endif
			right: 40upx;
			z-index: 9;
		}
	}
</style>
