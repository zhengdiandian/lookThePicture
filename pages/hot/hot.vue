<template>
	<view class="index">
		<block v-for="(data, index) in lists" :key="index">
			<view class="row">
				<view class="card card-list2" v-for="(item, i) in data" :key="i" @click="goDetail(item)">
					<image class="card-img card-list2-img" :src="item.img_src"></image>
					<span class="card-num-view card-list2-num-view">{{ item.img_num }}P</span>
					<view class="card-bottm row">
						<view class="car-title-view row">
							<span class="card-title card-list-title">{{ item.title }}</span>
						</view>
						<view class="card-share-view" @click.stop="share(item)"></view>
					</view>
				</view>
			</view>
		</block>
		<span class="loadMore">加载中...</span>
	</view>
</template>
<script>
export default {
	data() {
		return {
			refreshing: false,
			lists: [],
			fetchPageNum: 1
		};
	},
	methods: {
		getData() {
			uni.request({
				url: `${this.$serverUrl}/api/picture/posts.php?page=${this.refreshing ? 1: this.fetchPageNum}`,
				method: 'GET',
				data: {},
				success: (res) => {
					if (res.statusCode !== 200) {
						console.error('请求失败', res)
					} else {
						if(this.refreshing && res.data[0].id === this.lists[0][0]){
							uuni.showToast({
								title: '以是最新',
								mask: false,
								duration: 1500
							});
						}
						let list = [],
						lists = []
						res.data.forEach((item, index) => {
							// let index = Math.floor(index /2)
							list.push(item)
							if(index % 2 == 1) {
								lists.push(list)
								list = []
							}
						})
						if(this.refreshing) {
							this.refreshing = false
							uni.stopPullDownRefresh();
							this.lists = lists
							this.fetchPageNum = 2
						} else {
							this.lists = this.lists.concat(lists)
							this.fetchPageNum++
						}

					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		goDetail(item) {
			uni.navigateTo({
				url: '../detail/detail?data=' + encodeURIComponent(JSON.stringify(item)),
				success: res => {},
				fail: () => {},
				complete: () => {}
			});
		},
		share(item) {
			if(this.provider.length === 0) {
				uni.showToast({
					title: '当前环境无分享渠道'
				})
				return
			}
			let itemList = this.provider.map(item => item.name)
			uni.showActionSheet({
				itemList,
				success(res) {
					uni.share({
						provider: this.provider[res.tapIndex],
						type: 0,
						title: 'lookThepicture',
						href: '',
						imageUrl: '',
						success: res => {},
						fail: () => {
							uni.showToast({
								title: '分享失败',
								mask: false,
								duration: 1500
							});
						},
						complete: () => {}
					});
				}
			})
		}
	},
	onLoad() {
		this.getData()
		uni.getProvider({
			service: 'share',
			success: (res) => {
				let arr= []
				res.provider.forEach(item => {
					switch (item){
						case 'weixin':
							arr.push({
								name: '分享到微信朋友',
								id: item
							}),
							arr.push({
								name: '分享到朋友圈',
								type : 'WXSenceTimeLine'
							})
							break;
						case 'qq':
							arr.push({
								name: '分享到qq',
								id: item
							})
							break
						default:
							break;
					}
				})
			this.provider = arr
			}
		})
	},
	onPullDownRefresh () {
		this.refreshing = true
		this.getData()
	},
	onReachBottom()  {
		this.getData()
	},
	
};
</script>
<style></style>
