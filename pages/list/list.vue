<template>
	<view class="index">
		<block v-for="(list, index) in lists" :key="index">
			<view class="row">
				<view class="card card-list2" v-for="(item,i) in list" :key="i">
					<image class="card-img card-list2-img" :src="item.img_src"></image>
					<text class="card-num-view card-list2-title">{{item.img_num}}P</text>
					<view class="card-bottm row">
						<view class="card-title-view row">
							<text class="card-title card-list2-title">{{item.title}}</text>
						</view>
						<view class="card-share-view"></view>
					</view>
				</view>
			</view>
		</block>
	</view>
</template>
<script>
export default {
	data() {
		return {
			refreshing: false,
			id: 0,
			loadModeText: '加载中...',
			lists: [],
			fetchpageNum: 0
		};
	},
	methods: {
		getdata() {
			uni.request({
				url: `${this.$serverUrl}/api/picture/list.php?type=${this.id}`,
				method: 'GET',
				data: {},
				success: res => {
					if(res.statusCode !== 200){
						console.error('请求失败')
					} else {
						if(this.refreshing && res.data.data[0].id === this.lists[0][0].id) {
							uni.showToast({
								title: '没有新的数据啦!',
								mask: false,
								duration: 1500
							});
							this.refreshing = false
							uni.stopPullDownRefresh();
						}
						let list = [];
						let lists = []
						let data = res.data.data
						data.forEach((item,index) =>{
							let num = Math.floor(index / 2)
							list.push(data[index])
							if(index %2 == 1){
								lists.push(list)
								list = []
							}
						})
						if(this.refreshing) {
							this.refreshing = false
							uni.stopPullDownRefresh();
							this.lists = lists
							this.fetchpageNum = 2
						} else {
							this.lists = this.lists.concat(lists)
							this.fetchpageNum++
						}
					}
				},
				fail: () => {},
				complete: () => {}
			});
		}
	},
	onPullDownRefresh() {
		this.refreshing = true
		this.getdata()
	},
	onReachBottom() {
		if(this.fetchpageNum > 4) {
			this.loadModeText = '没有更多了'
			return
		}
		this.getdata()
	},
 	onLoad(e) {
		uni.setNavigationBarTitle({
			title: `专题: ${e.type} `
		})
		this.id = e.id
		setTimeout(() => {
			this.getdata()
		},300)
		uni.getProvider({
			service: "share",
			success: (e) =>{
				let	data = []
				e.provider.forEach(item =>{
					switch (item) {
						case 'weixin': 
							data.push({
								name: '分享到微信好友',
								id: 'weixin'
							})
							data.push({
								name: '分享到朋友圈',
								id: 'WXSencetimeLine'
							})
						case 'qq': 
							data.push({
								name: '分享到QQ',
								id: 'qq'
							})
					}
				})
				this.providerList = data
			} 
		})
		
	}
};
</script>
<style></style>
