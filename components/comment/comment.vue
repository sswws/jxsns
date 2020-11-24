<template>
	<view>
		<!-- 评论列表 -->
		<view class="comments">
			<view class="title gohere" id="gohere" ref="gohere">最新评论</view>
			<view class="no-comment" v-if="commintsList.length == 0">暂无评论</view>
			<view v-for="(commentItem, index) in commintsList" :key="index" class="one-comment">
				<!-- 一级评论相关 -->
				<view class="commenter">
					<view class="info">
						<view class="left">
							<u-avatar size="50" class="avatar" :src="!!commentItem.user.avatar ? commentItem.user.avatar.url : '' " />
							<view class="name">{{ commentItem.user.name }}</view>
						</view>
					</view>
					<view class="content">{{ commentItem.body }}</view>
					<view class="uptime">{{ commentItem.created_at | timeFormate }} 评论</view>
				</view>
			</view>
		</view>

    </view>
</template>

<script>
import timeFrom from "@/tools/timeFrom.js";
export default {
  props: {
    oneInfo: Object,
    type: String,
  },
  data() {
    return {
      // 当前动态评论列表详情
      commintsList: []
    };
  },
  async created() {
	  console.log(this.oneInfo)
    this.getCommentsList()
  },
  methods: {
    // 获取评论列表
    async getCommentsList() {
      if (this.type === "feed") {
        // 获取当前动态评论列表信息
        let res = await this.$u.api.getFeedComments({ id: this.oneInfo.id })
        this.commintsList = res.data.comments
      } else {
        // 获取当前资讯评论列表信息
        let res= await this.$u.api.getNewComments({ id: this.oneInfo.id })
		this.commintsList = res.data.comments
      }

    }
  },
  // 过滤器
  filters: {
    timeFormate(timeDate) {
      let Time = new Date(timeDate);
      let timestemp = Time.getTime();
      let t = timeFrom(timestemp, "yyyy年mm月dd日");
      return t;
    },
  },
};
</script>

<style lang="scss" scoped>
	.comments {
		padding: 20upx 20upx 200upx;

		.title {
			font-weight: bolder;
			font-size: 32upx;
			margin-bottom: 40upx;
		}

		.no-comment {
			text-align: center;
			margin: 120upx 0 280upx;
			color: #666;
		}

		.one-comment {

			&.last-comment {
				border-bottom: none;
			}

			.commenter {
				.info {
					display: flex;
					flex-direction: row;
					flex-wrap: wrap;
					justify-content: space-between;
					align-items: center;
					align-content: center;
					margin-top: 20upx;

					.left {
						display: flex;
						flex-direction: row;
						flex-wrap: wrap;
						justify-content: flex-start;
						align-items: center;
						align-content: center;

						.avatar {
							margin-right: 20upx;
							margin-top: 10upx;
						}

						.name {
							font-size: 24upx;
							font-weight: bolder;
							color: #1f2046;
						}

					}

				}
				.content{
					color: #666;
					font-size: 30upx;
					line-height: 50upx;
					margin-left:70upx;
					margin-top: 10upx;
					padding-right: 20upx;
				}
				.uptime{
					color: #999;
					font-size: 20upx;
					margin-left:70upx;
					margin-top: 10upx;
					padding-bottom: 20upx;
					border-bottom: 1upx solid #f2f2f2;
				}
			}
		}
	}
</style>
