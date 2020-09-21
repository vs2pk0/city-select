<template>
	<view>
		<button type="default" @click="goPage('citys')">城市示例</button>
		<button type="default" @click="goPage('address-list')">通讯录示例</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {};
		},
		methods: {
			goPage(str) {
				console.log(`/pages/${str}/${str}`)
				uni.navigateTo({
					url: `/pages/${str}/${str}`
				});
			}
		}
	};
</script>

<style lang="scss"></style>
