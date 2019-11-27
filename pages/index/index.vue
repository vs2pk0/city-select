<template>
	<view>
		<city-select
			@cityClick="cityClick"
			:formatName="formatName"
			:activeCity="activeCity"
			:hotCity="hotCity"
			:obtainCitys="obtainCitys"
			:isSearch="true"
			ref="citys"
		></city-select>
	</view>
</template>

<script>
import citys from './citys.js'
console.log(citys.length)
import citySelect from '@/components/city-select/city-select.vue'
export default {
	data() {
		return {
			//需要构建索引参数的名称（注意：传递的对象里面必须要有这个名称的参数）
			formatName: 'title',
			//当前城市
			activeCity: {
				id: 1,
				title: '南京市'
			},
			//热门城市
			hotCity: [
				{
					id: 0,
					title: '南京市'
				},
				{
					id: 1,
					title: '南京市'
				}
			],
			//显示的城市数据
			obtainCitys: [
				{
					id: 0,
					title: '南京'
				},
				{
					id: 1,
					title: '北京'
				},
				{
					id: 2,
					title: '天津'
				},
				{
					id: 3,
					title: '东京'
				}
			]
		}
	},
	components: {
		citySelect
	},
	onLoad() {
		//动态更新数据
		setTimeout(() => {
			//修改数据格式
			this.formatName = 'cityName'
			//修改当前城市
			this.activeCity = {
				cityName: '南京',
				cityCode: 110100
			}
			//修改热门城市
			this.hotCity = [
				{
					cityName: '南京',
					cityCode: 110100
				},
				{
					cityName: '北京',
					cityCode: 110102
				}
			]
			//修改构建索引数据
			this.obtainCitys = citys
			uni.showToast({
				icon: 'none',
				title: '更新数据成功',
				// #ifdef MP-WEIXIN
				duration: 3000,
				// #endif
				mask: true
			})
		}, 5000)
	},
	methods: {
		cityClick(item) {
			uni.showToast({
				icon: 'none',
				title: 'item: ' + JSON.stringify(item),
				// #ifdef MP-WEIXIN
				duration: 3000,
				// #endif
				mask: true
			})
		}
	}
}
</script>

<style></style>
