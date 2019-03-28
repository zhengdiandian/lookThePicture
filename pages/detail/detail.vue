<template>
	<view class="index">
		<swiper @change="swiperChange" :style="{height:screenHeight + 'px'}" @click="preImg(index)">
			<swiper-item v-for="(item, index) in data" :key="index" @click="preImg(index)">
				<image :src="item" alt="" mode="widthFix"></image>
			</swiper-item>
		</swiper>
		<!-- #ifndef H5 -->
		<view class="detail-btn-view">
			<view class="download" @click="dowmload"></view>
			<!-- #ifdef APP-PLUS -->
			<view v-if="showBtn" class="setting" @click="setting">设置为壁纸</view>
			<!-- #endif -->
			<view class="collect" @click="collect"></view>
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgShow: false,
				index: 0,
				showBtn: false,
				screenHeight: 0,
				imgLength: 0,
				providerList: [],
				data: [],
				detailDec: ''
			}
		},
		onErro(e) {
			console.log('出错',e)
		},
		onLoad (e) {
			this.init(e)
		},
		onNavigationBarButtonTap(e) {
			if(e.index === 1) {
				this.collect()
				return
			}
			if(this.providerList.length === 0) {
				uni.showToast({
					title: '当前环境无分享渠道',
					mask: false,
					duration: 1500
				});
				return
			}
			let itemList = this.providerList.map(val => val.name)
			uni.showActionSheet({
				itemList,
				success: (res) => {
					uni.share({
						provider: this.providerList[res.tapIndex].id,
						type: 0,
						title: 'lookThePicture',
						href: 'https://www.baidu.com',
						imageUrl: this.data[this.index],
						success: res => {},
						fail: (e) => {
							uni.showModal({
								title: e.errMsg,
								content: e.errMsg,
								showCancel: false,
								cancelText: '',
								confirmText: '',
								success: res => {},
								fail: () => {},
								complete: () => {}
							});
						},
						complete: () => {}
					});
				}
			})
		},
		methods: {
			init(e) {
				let data = JSON.parse(decodeURIComponent(e.data))
				// #ifdef APP-PLUS
				if(plus.os.name === "Android") {	
					this.showBtn = true
				}
				// #endif
				this.screenHeight = uni.getSystemInfoSync().windowHeight
				this.detailDec = e.data
				this.imgLength = data.img_num
				this.data.push(data.img_src)
				this.getData(data.id)
				uni.setNavigationBarTitle({
					title: `1/${this.imgLength}`
				})
				uni.getProvider({
				service: "share",
				success: (e) => {
					let data = []
					for (let i = 0; i < e.provider.length; i++) {
						switch (e.provider[i]) {
							case 'weixin':
								data.push({
									name: '分享到微信好友',
									id: 'weixin'
								})
								data.push({
									name: '分享到微信朋友圈',
									id: 'weixin',
									type: 'WXSenceTimeline'
								})
								break;
							case 'qq':
								data.push({
									name: '分享到QQ',
									id: 'qq'
								})
								break;
							default:
								break;
						}
					}
					this.providerList = data;
				},
				fail: (e) => {
					console.log("获取登录通道失败", e);
				}
			});
			},
			getData(params) {
				uni.request({
					url: `${this.$serverUrl}/api/picture/detail.php?id=${params}`,
					method: 'GET',
					data: {},
					success: res => {
						if(res.data.code !== 0) {
							uni.showModal({
								title: '',
								content: '请求失败' + res.data.msg,
								showCancel: false,
								cancelText: '',
								confirmText: '',
								success: res => {},
								fail: () => {},
								complete: () => {}
							});
							return
						}
						this.data = this.data.concat(res.data.data)
					},
					fail: () => {
						uni.showModal({
							title: '',
							content: '请求失败请重试',
							showCancel: false,
							cancelText: '',
							confirmText: '',
							success: res => {},
							fail: () => {},
							complete: () => {}
						});
					},
					complete: () => {}
				});
				
			},
			swiperChange(e) {
				this.index = e.detail.current
				uni.setNavigationBarTitle({
					title: `${this.index + 1}/${this.imgLength}`
				})
			},
			dowmload() {
				uni.downloadFile({
					url: this.data[this.index],
					success:(e) =>{
						uni.saveImageToPhotosAlbum({
							filePath: e.tempFilePath,
							success: () => {
								uni.showToast({
									title: '已保存',
									mask: false,
									duration: 1500
								});
							},
							fail: () => {
								uni.showToast({
									title: '下载失败',
									mask: false,
									duration: 1500
								});
							}
						})
					}
				})
			},
			collect() {
				uni.showToast({
					title: '点击收藏'
				})
			},
			// #ifdef APP-PLUS
			setting() {
				uni.showToast({
					title: '正在设置壁纸',
					// mask: false,
				});
				setTimeout(() => {
					try{
						var Main = plus.android.runtimeMainActivity()
						var WallpaperManager = plus.android.importClass("android.app.WallpaperManager")
						var wallpaperManager =WallpaperManager.getInstance(Main)
						plus.android.importClass(wallpaperManager)
						var Bitmapfactory = plus.android.importClass('android.graphics.BitmapFactory')
					}catch(e){
						console.log('216hang',e)
						//TODO handle the exception
					}
					
					uni.downloadFile({
						url: this.data[this.index],
						success: (e) => {
							var filePath = e.tempFilePath.replace('file://', '')
							console.log('path',JSON.stringify(e))
							filePath = plus.io.convertAbsoluteFileSystem(filePath)
							var bitmap = Bitmapfactory.decodeFile(filePath)
							try {
								wallpaperManager.setBitmap(bitmap)
								bitmap.recycle()
								uni.showToast({
									title: '设置壁纸成功',
									mask: false,
									duration: 1500
								});
							} catch (e) {
								console.error('err',e.errMsg)

								uni.showToast({
									title: '设置壁纸失败',
									mask: false,
									duration: 1500
								});
							}
						},
						fail: (e) => {
							console.error('err',e)
							uni.showToast({
								title: '设置壁纸失败',
								mask: false,
								duration: 1500
							});
						}
					})
				}, 100);
			},
			// #endif
			preImg(index) {
				if(this.imgShow) return
				this.imgShow = true
				setTimeout(() => {
					this.imgShow = false
				}, 1000)
				setTimeout(() => {
					uni.previewImage({
						current: this.data[index],
						urls: this.data
					})
				}, 150)
			}
		},
	}
</script>

<style>
	page {
		background-color: #000;
        height: 100%;
	}

	swiper {
		flex: 1;
		width: 750upx;
		background-color: #000;
		display: flex;
		flex-direction: column;
	}

	swiper-item {
		display: flex;
		align-items: center;
	}

	image {
		width: 750upx;
		height: 1125upx;
	}
	/* page{
		background-color: #000000;
		height: 100%;
	}
	swiper{
		display: flex;
		flex-direction: column;
		flex: 1;
		width: 100%;
		background-color: #000000;
	}
	swiper-item{
		
		display: flex;
		align-items: center;
	}
	image{
		width: 100%;
		height: 1125upx;
	} */
</style>
