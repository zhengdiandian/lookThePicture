<template>
	<view class="index">
		<view class="tags">
			<block v-for="(item, index) in data" :key="index">
				<view class="tag" @click="goList(item)">
					<img :src="item.icon" alt="" class="tag-img">
					<text class="tag-text">{{item.type}}</text>
				</view>
			</block>
		</view>
	</view>
</template>
<script>
export default {
	data() {
		return {
			data: [
				{
						type: '动物',
						id: 1,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '风景',
						id: 2,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '建筑',
						id: 3,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '美女',
						id: 4,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '汽车',
						id: 5,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '运动',
						id: 6,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '动物',
						id: 1,
						icon: 'http://placehold.it/150x150'
					},
					{
						type: '风景',
						id: 2,
						icon: 'http://placehold.it/150x150'
					}
			]
		};
	},
	methods: {
		goList(value) {
			uni.navigateTo({
				url: '../list/list?type=' + value.type + '&id=' + value.id,
				success: res => {},
				fail: () => {},
				complete: () => {}
			});
		}
	},
	onLoad() {}
};
</script>
<style></style>
