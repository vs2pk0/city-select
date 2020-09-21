<template>
	<!-- 城市选择-->
	<view class="city-select">
		<scroll-view scroll-y="true" class="city-select-main" id="city-select-main" :scroll-into-view="toView">
			<!-- 预留搜索-->
			<view class="city-serach" v-if="isSearch"><input @input="keyInput" :placeholder="placeholder" class="city-serach-input" /></view>
			<!-- 当前定位城市 -->
			<view class="hot-title" v-if="activeCity && !serachCity">当前定位城市</view>
			<view class="hot-city" v-if="activeCity && !serachCity">
				<view class="hot-item" @click="cityTrigger(activeCity)">{{ activeCity[formatName] }}</view>
			</view>
			<!-- 热门城市 -->
			<view class="hot-title" v-if="hotCity.length > 0 && !serachCity">热门城市</view>
			<view class="hot-city" v-if="hotCity.length > 0 && !serachCity">
				<template v-for="(item, index) in hotCity">
					<view :key="index" @click="cityTrigger(item, 'hot')" class="hot-item">{{ item[formatName] }}</view>
				</template>
			</view>
			<!-- 城市列表(搜索前) -->
			<view class="citys" v-if="!serachCity">
				<view v-for="(city, index) in sortItems" :key="index" v-show="city.isCity" class="citys-row">
					<view class="citys-item-letter" :id="'city-letter-' + (city.name === '#' ? '0' : city.name)">{{ city.name }}</view>
					<view class="citys-item" v-for="(item, inx) in city.citys" :key="inx" @click="cityTrigger(item)">{{ item.cityName }}</view>
				</view>
			</view>
			<!-- 城市列表(搜索后)  -->
			<view class="citys" v-if="serachCity">
				<view v-for="(item, index) in searchDatas" :key="index" class="citys-row">
					<view class="citys-item" :key="inx" @click="cityTrigger(item)">{{ item.name }}</view>
				</view>
			</view>
		</scroll-view>
		<!-- 城市选择索引-->
		<view class="city-indexs-view" v-if="!serachCity">
			<!-- #ifndef MP-WEIXIN -->
			<view class="city-indexs" id="index-bar">
				<view v-for="(cityIns, index) in handleCity" class="city-indexs-text" v-show="cityIns.isCity" :key="index" @click="cityindex(cityIns.forName)">
					{{ cityIns.name }}
				</view>
			</view>
			<!-- #endif -->
			<!-- #ifdef MP-WEIXIN -->
			<view class="city-indexs" @touchstart="tStart" @touchend="tEnd" @touchmove.stop="tMove" id="index-bar">
				<view v-for="(cityIns, index) in handleCity" class="city-indexs-text" v-show="cityIns.isCity" :key="index" @click="cityindex(cityIns.forName)">
					{{ cityIns.name }}
				</view>
			</view>
			<!-- #endif -->

		</view>
		<!--选择显示-->

		<!-- #ifdef MP-WEIXIN -->
		<view v-show="!hidden" class="index-toast">
			{{listCurID}}
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
	import citySelect from './citySelect.js';
	export default {
		props: {
			//查询提示文字
			placeholder: {
				type: String,
				default: '请输入城市名称'
			},
			//传入要排序的名称
			formatName: {
				type: String,
				default: 'cityName'
			},
			//当前定位城市
			activeCity: {
				type: Object,
				default: () => null
			},
			//热门城市
			hotCity: {
				type: Array,
				default: () => []
			},
			//城市数据
			obtainCitys: {
				type: Array,
				default: () => []
			},
			//是否有搜索
			isSearch: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				itemHeight: 0, //索引item高度
				hidden: true, //滑动显示
				listCurID: '', //滑动显示的字母
				toView: 'city-letter-Find', //锚链接 初始值
				cityindexs: [], // 城市索引
				activeCityIndex: '', // 当前所在的城市索引
				handleCity: [], // 处理后的城市数据
				serachCity: '', // 搜索的城市
				cityData: []
			};
		},
		computed: {
			/**
			 * @desc 城市列表排序
			 * @return  Array
			 */
			sortItems() {
				for (let index = 0; index < this.handleCity.length; index++) {
					if (this.handleCity[index].isCity) {
						let cityArr = this.handleCity[index].citys;
						cityArr = cityArr.sort(function(a, b) {
							var value1 = a.unicode;
							var value2 = b.unicode;
							return value1 - value2;
						});
					}
				}
				return this.handleCity;
			},
			/**
			 * @desc 搜索后的城市列表
			 * @return Array
			 */
			searchDatas() {
				var searchData = [];
				for (let i = 0; i < this.cityData.length; i++) {
					if (this.cityData[i][this.formatName].indexOf(this.serachCity) !== -1) {
						searchData.push({
							oldData: this.cityData[i],
							name: this.cityData[i][this.formatName]
						});
					}
				}
				return searchData;
			}
		},
		created() {
			// 初始化城市数据
			this.cityData = this.obtainCitys;
			this.initializationCity();
			this.buildCityindexs();
		},
		watch: {
			obtainCitys(newData) {
				this.updateCitys(newData);
			}
		},
		// #ifdef MP-WEIXIN
		mounted() {
			this.getIndexsHeight();
		},
		// #endif
		methods: {
			// #ifdef MP-WEIXIN
			//获取索引高度
			getIndexsHeight() {
				//获取索引高度
				let that = this;
				const query = uni.createSelectorQuery().in(this);
				query.select('#index-bar').boundingClientRect(data => {
					// console.log("得到布局位置信息", JSON.stringify(data));
					// console.log("节点离页面顶部的距离为", data.top);
					that.barTop = data.top
					const cityIndexName = this.handleCity.filter(item => item.isCity);
					this.itemHeight = data.height / cityIndexName.length;
				}).exec();
			},
			//滑动选择Item
			tMove(e) {
				this.hidden = false
				this.listCurID = '';
				const cityIndexName = this.handleCity.filter(item => item.isCity);
				let y = e.touches[0].clientY,
					offsettop = this.barTop,
					that = this;
				//判断选择区域,只有在选择区才会生效
				if (y > offsettop) {
					let num = parseInt((y - offsettop) / this.itemHeight);
					// console.log(num, cityIndexName.length)
					if (num > cityIndexName.length) {
						this.hidden = true
						this.listCurID = '';
					} else {
						this.hidden = false
						this.listCurID = cityIndexName[num].name
					}
				} else {
					this.listCurID = '';
					this.hidden = true
				}
			},

			//触发全部开始选择
			tStart() {
				if (this.listCurID) {
					this.hidden = false
					this.listCurID = '';
				}
			},

			//触发结束选择
			tEnd() {
				this.hidden = true;
				this.toView = 'city-letter-' + (this.listCurID == "#" ? "0" : this.listCurID);
				this.listCurID = '';
			},
			// #endif
			/**
			 * @desc 初始化
			 */
			updateCitys(data) {
				if (data && data.length) {
					this.cityData = data;
					this.initializationCity();
					this.buildCityindexs();
					// #ifdef MP-WEIXIN
					setTimeout(() => {
						this.getIndexsHeight();
					}, 500);
					// #endif
				}
			},
			/**
			 * @desc 监听输入框的值
			 */
			keyInput(event) {
				this.serachCity = event.detail.value;
			},
			/**
			 * @desc 初始化城市数据
			 * @return undefind
			 */
			initializationCity() {
				this.handleCity = [];
				const cityLetterArr = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R',
					'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '#'
				];
				for (let index = 0; index < cityLetterArr.length; index++) {
					this.handleCity.push({
						name: cityLetterArr[index],
						isCity: false, // 用于区分是否含有当前字母开头的城市
						citys: [], // 存放城市首字母含是此字母的数组
						forName: 'city-letter-' + (cityLetterArr[index] == '#' ? '0' : cityLetterArr[index]) //label的绑定
					});
				}
			},
			/**
			 * @desc 得到城市的首字母
			 * @param str String
			 */
			getLetter(str) {
				return citySelect.getFirstLetter(str[0]);
			},
			/**
			 * @desc 构建城市索引
			 * @return undefind
			 */
			buildCityindexs() {
				this.cityindexs = [];
				for (let i = 0; i < this.cityData.length; i++) {
					// 获取首字母
					const cityLetter = this.getLetter(this.cityData[i][this.formatName]).firstletter;
					// 获取当前城市首字母的unicode，用作后续排序
					const unicode = this.getLetter(this.cityData[i][this.formatName]).unicode;

					const index = this.cityIndexPosition(cityLetter);
					if (this.cityindexs.indexOf(cityLetter) === -1) {
						this.handleCity[index].isCity = true;
						this.cityindexs.push(cityLetter);
					}

					this.handleCity[index].citys.push({
						cityName: this.cityData[i][this.formatName],
						unicode: unicode,
						oldData: this.cityData[i]
					});
				}
			},
			/**
			 * @desc 滑动到城市索引所在的地方
			 * @param id String 城市索引
			 */
			cityindex(id) {
				this.toView = id;
			},
			/**
			 * @desc 获取城市首字母的unicode
			 * @param letter String 城市索引
			 */
			cityIndexPosition(letter) {
				if (!letter) {
					return '';
				}
				const ACode = 65;
				return letter === '#' ? 26 : letter.charCodeAt(0) - ACode;
			},
			/** @desc 城市列表点击事件
			 *  @param Object
			 */
			cityTrigger(item) {
				// 传值到父组件
				this.$emit('cityClick', item.oldData ? item.oldData : item);
			}
		}
	};
</script>

<style lang="scss">
	//宽度转换vw
	@function vww($number) {
		@return ($number / 375) * 750+rpx;
	}

	/* #ifndef MP-ALIPAY */
	.index-toast {
		position: fixed;
		top: 0;
		right: vww(40);
		bottom: 0;
		background: rgba(0, 0, 0, 0.5);
		width: vww(50);
		height: vww(50);
		border-radius: 10px;
		margin: auto;
		color: #fff;
		line-height: vww(50);
		text-align: center;
		font-size: vww(16);
	}

	/* #endif */

	view {
		box-sizing: border-box;
	}

	.city-serach {
		width: 100%;
		color: #4a4a4a;
		padding: 0 vww(10);

		&-input {
			margin: vww(10) 0;
			height: vww(40);
			line-height: vww(40);
			font-size: vww(14);
			padding: 0 vww(5);
			border: 1px solid #4d8cfd;
			border-radius: 3px;
		}
	}

	.city-select-main {
		position: relative;
		// overflow: scroll;
		// -webkit-overflow-scrolling: touch;
		width: 100%;
		height: 100%;
		background: #f6f5fa;
		// overflow-y: auto;
	}

	.city-select {
		position: relative;
		width: 100vw;
		height: 100vh;
		background: #f6f5fa;

		// 热门城市
		.hot-title {
			padding-left: vww(23);
			width: 100vw;
			font-size: 14px;
			line-height: vww(40);
			color: #9b9b9b;
		}

		.hot-city {
			padding-left: vww(23);
			padding-right: vww(20);
			overflow: hidden;
			width: 100vw;

			.hot-item {
				float: left;
				padding: 0 vww(5);
				margin-right: vww(16);
				margin-bottom: vww(6);
				overflow: hidden;
				width: vww(100);
				height: vww(31);
				font-size: 14px;
				text-align: center;

				display: -webkit-box;
				-webkit-box-orient: vertical;
				-webkit-line-clamp: 1;

				line-height: vww(31);
				color: #4a4a4a;
				background: #fff;
				border: 1px solid #ebebf0;

				&:nth-child(3n) {
					margin-right: 0;
				}
			}

			.hot-hidden {
				display: none;
				margin-right: 0;
			}
		}

		.citys {
			.citys-row {
				padding-left: vww(18);
				width: 100%;
				font-size: 14px;
				background: #fff;

				.citys-item-letter {
					margin-left: vww(-18);
					padding-left: vww(18);
					margin-top: -1px;
					width: 100vw;
					line-height: vww(30);
					color: #9b9b9b;
					background: #f6f5fa;
					border-top: none;
				}

				.citys-item {
					width: 100%;
					line-height: vww(50);
					color: #4a4a4a;
					border-bottom: 1px solid #ebebf0;

					&:last-child {
						border: none;
					}
				}
			}
		}

		.city-indexs-view {
			position: absolute;
			right: 0;
			top: 0;
			z-index: 999;
			display: flex;
			width: vww(20);
			height: 100%;
			text-align: center;

			.city-indexs {
				width: vww(20);
				text-align: center;
				vertical-align: middle;
				align-self: center;

				.city-indexs-text {
					// margin-bottom: vww(10);
					width: vww(20);
					padding: vww(5) 0;
					font-size: 12px;
					color: #4d8cfd;

					// &:last-child {
					// 	margin-bottom: 0;
					// }
				}
			}
		}
	}
</style>
