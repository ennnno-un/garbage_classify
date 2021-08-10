<template>
	<view class="container">
		<uni-popup ref="popup"></uni-popup>
		<view class="info">
			<view class="avatar">
				<image :src="user_info.avatarUrl"></image>
				<i class="iconfont icon-nan" v-if="user_info.gender == 1"></i>
				<i class="iconfont icon-nv" v-else></i>
			</view>
			<view class="name">
				{{user_info.nickName}}
			</view>
		</view>
		<uni-table border stripe emptyText="暂无更多数据" v-if="show">
			<uni-tr class="head">
				<uni-th align="center" width="300r">图片</uni-th>
				<uni-th align="center" width="225r">结果</uni-th>
				<uni-th align="center" width="225r">操作</uni-th>
			</uni-tr><br>
			<uni-tr>
				<uni-td align="center" width="300r"></uni-td>
				<uni-td align="center" width="225r">厨余垃圾<br>
				水果果皮</uni-td>
				<uni-td align="center" width="225r">
					<view @click="deleteTable()">
						删除
					</view>
				</uni-td>
			</uni-tr>
		</uni-table>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				user_info: {
					id: "",
					avatarUrl: "",
					gender: "",
					nickName: ""
				},
				show: true
			}
		},
		onLoad(option) {
			this.$refs.popup.open();
			  const eventChannel = this.getOpenerEventChannel()
			  // 监听acceptDataFromOpenerPage事件，获取上一页面通过eventChannel传送到当前页面的数据
			  eventChannel.on('acceptDataFromOpenerPage', (data) => {
					console.log(data.id)
					console.log(data.data);
			    this.user_info.id = data.id ? data.id : "";
					this.user_info.avatarUrl = data.data.avatarUrl ? data.data.avatarUrl : "";
					this.user_info.gender = data.data.gender ? data.data.gender : "";
					this.user_info.nickName = data.data.nickName ? data.data.nickName : "";
			  })
		},
		methods: {
			deleteTable() {
				uni.showToast({title: "删除成功", icon: "none", mask: "true"})
				this.show = false;
			}
		}
	}
</script>

<style lang="scss">
	.container {
		.info {
			position: relative;
			width: 100%;
			height: 280rpx;
			background-color: #B5B8E6;
			.avatar {
				width: 150rpx;
				height: 150rpx;
				position: absolute;
				left: 50%;
				margin-left: -75rpx;
				image {
					width: 150rpx;
					height: 150rpx;
					border-radius: 50%;
				}
				.iconfont {
					width: 36rpx;
					height: 36rpx;
					background-color: #FFF;
					border-radius: 50%;
					position: absolute;
					right: 0;
					bottom: 0;
					font-size: 36rpx;
				}
				.icon-nan {
					color: #35A4FE;
				}
				.icon-nv {
					color: #FA4E8D;
				}
			}
			.name {
				width: 100%;
				font-weight: 700;
				font-size: 36rpx;
				text-align: center;
				position: absolute;
				top: 200rpx;
				color: #FFF;
			}
		}
	}
</style>
