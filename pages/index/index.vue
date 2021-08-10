<template>
	<view class="container">
		<button type="default" class="btn" plain=true hover-class="none" style="border: none;" @click="play()">
			<image src="../../static/images/button.png"></image>
		</button>
		<uni-popup ref="popupvideo" type="center">
			<view class="pop-video">
				<view class="video">
					<txv-video vid="n0354rivuel" playerid="txv1"></txv-video>
				</view>
			</view>
		</uni-popup>
		<uni-grid :column="2" :showBorder="false" class="main" highlight="false" @change="open">
			<uni-grid-item class="block" index="2"><i class="iconfont icon-chuyulaji"></i></uni-grid-item>
			<uni-grid-item class="block" index="3"><i class="iconfont icon-kehuishouwu"></i></uni-grid-item>
			<uni-grid-item class="block" index="4"><i class="iconfont icon-youhailaji"></i></uni-grid-item>
			<uni-grid-item class="block" index="1"><i class="iconfont icon-qitalaji"></i></uni-grid-item>
		</uni-grid>
		<uni-popup ref="popup" type="center" animation="false">
			<view class="pop">
				<i class="iconfont icon-guanbi close" @click="close"></i>
				<image :src="image" class="image"></image>
				<view class="title">{{category}}</view>
				<view class="info">
					{{info}}
				</view>
			</view>
		</uni-popup>
		<view class="foot">
			<view class="preview" @click="catchImage()">
				<i class="iconfont icon-paizhao"></i>
			</view>
			<view class="user" @click="toUser()">
				<i class="iconfont icon-user"></i>
			</view>
		</view>
		<uni-popup ref="popupresult" type="center">
			<view class="img-container">
				<image :src="img" class="img"></image>
			</view>
			<view class="popup-result">
				<view class="result" v-if="name">
					<view class="tag1">
						识别出图中的物品为:<br>
						{{name}}
					</view>
					<!-- 无法识别，请返回后重新上传图片 -->
					{{tag}}
				</view>
				<view class="result" v-else>
					无法识别
				</view>
				<view class="tip" v-if="tip">
					(图片不够清晰，预测结果有些偏差哦~)
				</view>
				<button class="button" type="default" @click="this.$refs.popupresult.close()">返回</button>
				<button class="button" type="default" @click="repredict(img)">重新预测</button>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				index: "0",
				image: "data:image/jpg;base64,",
				category: "",
				info: "",
				img: null,
				result: "",
				name: "",
				tag: "",
				tip: false,
				wechat_id: "",
				user_info: {}
			}
		},
		created() {
		},
		mounted() {
			// this.$refs.popupresult.open();
		},
		methods: {
			play(){
				this.$refs.popupvideo.open();
			},
			open(e) {
				this.index = e.detail.index;
				uni.request({
					method: 'POST',
					url: 'http://127.0.0.1:5000/get_info',
					data: {"target": this.index},
					header: {'Content-Type': 'application/x-www-form-urlencoded'},
					success: (res) => {
						// console.log(res);
						this.image += res.data.data.image;
						this.category = res.data.data.category;
						this.info = res.data.data.info;
					},
					fail: (err) => {
						console.log(err);
					}
				});

				this.$refs.popup.open();
			},
			close() {
				this.$refs.popup.close();
			},
			catchImage() {
				if (this.wechat_id === "") {
					this.getUserProfile();
				}
				uni.chooseImage({
					count: 1,
					type: 'image',
					sourceType: ['album','camera'],
					success: res => {
						uni.showToast({title: "识别中", icon: "loading", mask: "true", duration: 15000});
						// console.log(res.tempFilePaths[0]);
						this.img = res.tempFilePaths[0];
						this.loadFile(this.img);
						this.$refs.popupresult.open()
					},
					fail: err => {
						if (err.errMsg === "chooseImage:fail cancel") {
							return;
						}
						uni.showToast({title: "请稍后再试", icon: "none", mask: "true"})
						console.log(err);
					}
				})
			},
			loadFile(img) {
				uni.uploadFile({
					url: 'http://127.0.0.1:5000/predict',
					filePath: img,
					name: 'pic',
					header: {'Content-Type': 'multipart/form-data'},
					timeout: 15000,
					success: res => {
						uni.hideToast();
						console.log(res.data);
						let data = JSON.parse(res.data);
						this.result = data.data[0];
						this.dealresult(this.result);
						console.log(this.result);
						this.$refs.popupresult.open();
					},
					fail: err => {
						uni.showToast({title: "内部服务器错误，请稍后再试", icon: "none", mask: "true"})
						console.log(err);
					}
				})
			},
			repredict(img) {
				this.$refs.popupresult.close();
				this.loadFile(img);
			},
			dealresult(result) {
				this.tip = false;
				this.tag = "";
				this.name = "";
				let c = result.c;
				if (c < 99.998) {
					return;
				}
				if (c < 1) {
					this.tip = true;
				}
				this.tag = result.name.split("/")[0];
				this.name = result.name.split("/")[1];
			},
			toUser() {
				if (this.wechat_id === "") {
					this.getUserProfile();
				}
				// if (this.wechat_id) {
				uni.navigateTo({
					url: "../user/user",
						success: function(res) {
							// 通过eventChannel向被打开页面传送数据
							res.eventChannel.emit('acceptDataFromOpenerPage', { id: this.wechat_id, data: this.user_info })
						}
				});
				// }
			},
			getUserProfile() {
				uni.getUserProfile({
					desc: "获取用户信息",
					success: (res) => {
						console.log(res);
						this.wechat_id = res.signature;
						this.user_info = res.userInfo;
					},
					fail: (err) => {
						uni.showToast({title: "获取用户信息失败，请稍后再试", icon: "none", mask: "true"})
						console.log(err);
						return;
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.container {
		width: 100%;
		button {
			box-sizing: content-box;
			background-color: transparent;
			width: 80%;
			height: 120rpx;
			margin-top: 80rpx;
			image {
				height: 100%;
				margin-left: -20rpx;
			}
		}
		.video {
			width: 750rpx;
			height: 500rpx;
		}
		.uni-grid-wrap {
			width: 680rpx;
			height: 680rpx;
			border-radius: 15%;
			margin: 60rpx auto;
			.iconfont {
				width: 320rpx;
				height: 320rpx;
				color: #B7F2FF;
				font-size: 280rpx;
				text-align: center;
			}
			.icon-chuyulaji {
				color: #239148;
				border: 5rpx dashed #239148;
				border-top-left-radius: 25%;
			}
			.icon-kehuishouwu {
				color: #186FB6;
				border: 5rpx solid #186FB6;
				border-top-right-radius: 25%;
				margin-left: 20rpx;
			}
			.icon-youhailaji {
				color: #BF1B22;
				border: 5rpx solid #BF1B22;
				border-bottom-left-radius: 25%;
				margin-top: 20rpx;
			}
			.icon-qitalaji {
				color: #787679;
				border: 5rpx dashed #787679;
				border-bottom-right-radius: 25%;
				margin-top: 20rpx;
				margin-left: 20rpx;
			}
		}
		.pop {
			width: 680rpx;
			height: 980rpx;
			border: 1rpx solid #404040;
			border-radius: 5%;
			background-color: #F0F0F0;
			overflow: auto;
			.close {
				position: absolute;
				right: 20rpx;
				top: 20rpx;
				color: #404040;
				font-size: 48rpx;
			}
			.image {
				width: 300rpx;
				height: 400rpx;
				border-radius: 15%;
				margin-top: 30rpx;
				margin-left: 190rpx;
			}
			.title {
				font-size: 48rpx;
				margin-top: 20rpx;
				margin-left: 20rpx;
				font-weight: 700;
				color: #2C405A;
			}
			.info {
				margin: 20rpx;
				font-size: 32rpx;
				color: #3F536E;
			}
		}
		.foot {
			// width: 200rpx;
			// height: 200rpx;
			position: absolute;
			left: 80rpx;
			bottom: 30rpx;
			// border-radius: 50%;
			// background-color: #B5B8E6;
			// line-height: 200rpx;
			// text-align: center;
			width: 600rpx;
			display: flex;
			justify-content: center;
			// position: absolute;
			// bottom: 30rpx;
			margin: 0 auto;
			.preview, .user {
				width: 200rpx;
				height: 200rpx;
				// position: absolute;
				// left: 275rpx;
				// bottom: 30rpx;
				// border-radius: 50%;
				background-color: #B5B8E6;
				line-height: 200rpx;
				text-align: center;
				i {
					font-size: 98rpx;
					color: #FFFFFF;
				}
			}
			.preview {
				border-top-left-radius: 15%;
				border-bottom-left-radius: 15%;
				// margin-right: 20rpx;
			}
			.user {
				border-top-right-radius: 15%;
				border-bottom-right-radius: 15%;
				margin-left: 20rpx;
			}
		}
		.img-container {
			width: 320rpx;
			height: 320rpx;
			background-color: #FFFFFF;
			margin-left: 220rpx;
			margin-bottom: 20rpx;
			.img {
				// position: absolute;
				max-width: 300rpx;
				max-height: 300rpx;
				margin-left: 10rpx;
				margin-top: 10rpx;
			}
		}
		
		.popup-result {
			width: 680rpx;
			height: 480rpx;
			background-color: #FFF;
			border: 8px solid #FFFF83;
			border-radius: 5%;
			.result {
				font-size: 72rpx;
				font-weight: 700;
				color: #55A532;
				text-align: center;
				margin-top: 20rpx;
				margin-bottom: 50rpx;
				.tag1 {
					color: #9A6E3A;
					text-align: center;
					font-size: 48rpx;
				}
			}
			.button {
				height: 80rpx;
				width: 540rpx;
				line-height: 80rpx;
				margin-top: 0;
			}
		}
	}
</style>