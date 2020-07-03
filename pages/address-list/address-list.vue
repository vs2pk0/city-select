<template>
	<view>
		<city-select
			placeholder="请输入联系人姓名"
			@cityClick="cityClick"
			:formatName="formatName"
			:obtainCitys="obtainCitys"
			:isSearch="true"
		></city-select>
	</view>
</template>

<script>
import phone from './phone.js';
import citySelect from '@/components/city-select/city-select.vue';
export default {
	data() {
		return {
			//需要构建索引参数的名称（注意：传递的对象里面必须要有这个名称的参数）
			formatName: 'name',
			//显示的城市数据
			obtainCitys: phone
		};
	},
	components: {
		citySelect
	},
	onLoad() {},
	methods: {
		cityClick(item) {
			uni.showToast({
				icon: 'none',
				title: 'item: ' + JSON.stringify(item),
				// #ifdef MP-WEIXIN
				duration: 3000,
				// #endif
				mask: true
			});
		}
	}
};
</script>

<style></style>
