<template>
	<view class="index">
		<view class="tags">
			<SearchBtn class="search-btn" placeholder="请输入关键字搜索图片" v-model="search" @searchClick="searchClick"></SearchBtn>
			<block v-for="(value, index) in data" :key="index">
				<view class="tag" @tap="goList(value)">
					<image class="tag-img" :src="value.icon"></image>
					<text class="tag-text">{{value.type}}</text>
				</view>
			</block>
		</view>
	</view>
</template>

<script>
	import SearchBtn from '../components/search-btn'

	export default {
		components: {
			SearchBtn
		},
		data() {
			return {
				data: [],
				search: ''
			}
		},
		methods: {
			goList(value) {
				uni.navigateTo({
					url: '../list/list?type=' + value.type + '&id=' + value.id + '&imageType=' + value.imageType
				})
			},
			async getImageData() {
				// 先根据微信名nickName分组
				const db = wx.cloud.database()
				const $ = db.command.aggregate
				let res =  await db.collection('imageItemList').aggregate()
				// group里必须要有一个id
				.group({
					_id: '$title',
					num: $.sum(1)
				})
				.end()
				// 再每组取一条数据
				if(res && res.list && res.list.length) {
					let list2 = []
					for(let i = 0;i< res.list.length;i++) {
						let item = res.list[i]
						let res2 = await db.collection('imageItemList').where({
							title: item._id
						}).limit(1).get()
						let data = res2.data[0]
						list2.push({
							type: data.title,
							id: data.title,
							icon: data.img_src,
							imageType: 1
						})
					}
					// 添加到分类
					this.data.push(...list2)

				}
			},
			async getUserImageData() {
				// 先根据微信名openId分组
				const db = wx.cloud.database()
				let ret = await wx.cloud.callFunction({
					name: 'getImageUser',
					data: {
					}
				})
				console.log(ret);
				let result = ret.result
				// 再每组取一条数据
				if(result && result.data && result.data.length) {
					let list2 = []
					for(let i = 0;i< result.data.length;i++) {
						let data = result.data[i]
						list2.push({
							type: data.nickName,
							id: data.openId,
							icon: data.avatarUrl,
							imageType: 2
						})
					}
					this.data = []
					// 添加到分类
					this.data.push(...list2)

				}
			},
			searchClick () {
				setTimeout(()=>{
					if(this.search) {
						uni.navigateTo({
							url: '/pages/img-search-list/img-search-list?search=' + this.search
						})
					} else {
						uni.showToast({
							title: '请输入关键字',
							icon: 'none',
						})
					}
				},10)
			}
		},
		async onPullDownRefresh() {
			await this.getUserImageData();
			await this.getImageData();
			uni.stopPullDownRefresh();
		},
		async onLoad() {
			await this.getUserImageData();
			await this.getImageData();
		}
	}
</script>

<style scoped lang="stylus">
.search-btn
	width 100%
	margin 10px
</style>
