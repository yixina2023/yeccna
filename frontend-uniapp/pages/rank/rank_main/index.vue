<template>
	<view class="new-users copy-data" :style="{height:pageHeight}">
		<view class="top" :style="colorStyle">
			<!-- #ifdef MP || APP-PLUS -->
			<view class="sys-head">
				<view class="sys-bar" :style="{height:sysHeight}"></view>
				<!-- #ifdef MP -->
				<view class="sys-title" style="color:#333">排行版</view>
				<!-- #endif -->
			</view>
			<!-- #endif -->
		</view>
		<view class="tab-container">
			<view class="tab-item" @click.stop="clickTab('积分榜')">
				积分榜
			</view>
			<view class="tab-item" @click.stop="clickTab('商品榜')">
				商品榜
			</view>
		</view>
		
		<view v-if="tabNow == '积分榜'">
			<view>
				<view class="itegral-rank-item" v-for="item,idx in integralRank">
					<view>{{idx + 1}}</view>
					<image class="avatar" :src='item.avatar' v-if="item.avatar"></image>
					<view>{{item.sum_integral}}</view>
					<view>{{item.nickname}}</view>
					
				</view>
			</view>
		</view>
		
		<view v-else-if="tabNow == '商品榜'" class="product-rank">
			<view class="product-list">
				<view v-for="item,idx in productList" :class="{'product-list-item': true, 'pli-border': idx == productSelected}" @click.stop="getProductUserRank(idx)">
					<view>
						<image class="product-image" :src='item.image' v-if="item.image"></image>
						<view class="product-name">{{item.store_name}}</view>	
					</view>
				</view>
			</view>
			
			<view class="product-user-rank">
				<view v-for="item,idx in productUserRank" class="pur-container">
					<view>{{idx + 1}}</view>
					<image class="avatar" :src='item.avatar' v-if="item.avatar"></image>
					<view class="nickname">{{item.nickname}}</view>
				</view>
			</view>
		</view>
		

		<tabBar v-if="!is_diy" :pagePath="'/pages/rank/rank_main/index'"></tabBar>
		<view class="foot" v-else-if="is_diy && newData.status && newData.status.status">
			<view class="page-footer" id="target" :style="{'background-color':newData.bgColor.color[0].item}">
				<view class="foot-item" v-for="(item,index) in newData.menuList" :key="index" @click="goRouter(item)">
					<block v-if="item.link == activeRouter">
						<image :src="item.imgList[0]"></image>
						<view class="txt" :style="{color:newData.activeTxtColor.color[0].item}">{{$t(item.name)}}</view>
					</block>
					<block v-else>
						<image :src="item.imgList[1]"></image>
						<view class="txt" :style="{color:newData.txtColor.color[0].item}">{{$t(item.name)}}</view>
					</block>
					<div class="count-num" v-if="item.link === '/pages/order_addcart/order_addcart' && cartNum > 0">
						{{cartNum}}
					</div>
				</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	let sysHeight = uni.getSystemInfoSync().statusBarHeight + 'px';
	import {
		wechatAuthV2,
		getNavigation,
		silenceAuth
	} from '@/api/public.js'
	import {
		getRankIntegral,
		getRankProductList,
		getRankProductUserRank
	} from '@/api/user.js';
	
	export default {
		data() {
			return {
				integralRank: [],
				productList: [],
				productSelected: -1,
				productUserRank: [],
				mpHeight: 0,
				sysHeight: sysHeight,
				tabNow: '积分榜',
				is_diy: uni.getStorageSync('is_diy'),
				newData: {},
				activeRouter: '',
			}
		},
		methods: {

			goRouter(item) {
				var pages = getCurrentPages();
				var page = (pages[pages.length - 1]).$page.fullPath;
				if (item.link == page) return
				uni.switchTab({
					url: item.link,
					fail(err) {
						uni.redirectTo({
							url: item.link
						})
					}
				})
			},
			clickTab(label) {
				var that = this;
				if (that.productList.length < 1 && this.tabNow != label && label == '商品榜') {
					// 获取商品榜
					getRankProductList().then(res=> {
						that.productList = res.data;
						// that.productList = [
						// 	{id: 8, image: "http://admin.yeccna.com/uploads/attach/2023/04/20230424/7fb7eb7292c8be4bd2dc5a1980cd5227.jpg",store_name: "护肤品"},
						// 	{id: 8, image: "http://admin.yeccna.com/uploads/attach/2023/04/20230424/7fb7eb7292c8be4bd2dc5a1980cd5227.jpg",store_name: "护肤品"},
						// 	{id: 8, image: "http://admin.yeccna.com/uploads/attach/2023/04/20230424/7fb7eb7292c8be4bd2dc5a1980cd5227.jpg",store_name: "护肤品"},
						// 	{id: 8, image: "http://admin.yeccna.com/uploads/attach/2023/04/20230424/7fb7eb7292c8be4bd2dc5a1980cd5227.jpg",store_name: "护肤品"},
						// 	{id: 8, image: "http://admin.yeccna.com/uploads/attach/2023/04/20230424/7fb7eb7292c8be4bd2dc5a1980cd5227.jpg",store_name: "护肤品"},
						// ];
						if (that.productList.length > 0) {
							that.productSelected = 0;
							getRankProductUserRank(that.productList[that.productSelected].id).then(res=> {
								that.productUserRank = res.data;
								// that.productUserRank = [
								// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
								// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
								// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
								// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
								// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
								// ];
							})
						}
					});
				}
				this.tabNow = label;
			},
			getProductUserRank(idx) {
				var that = this;
				that.productUserRank = [];
				if (that.productSelected == idx) {
					return;
				}
				
				that.productSelected = idx;
				getRankProductUserRank(that.productList[idx].id).then(res => {
					that.productUserRank = res.data;
					// that.productUserRank = [
					// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
					// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
					// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
					// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
					// 	{id: 1, nickname: "wx351674", avatar: "http://admin.yeccna.com/statics/system_images/default_avatar.jpeg"},
					// ];
				});
			}
		},
		onReady() {
			// let self = this
			// // #ifdef MP
			// let info = uni.createSelectorQuery().select(".sys-head");
			// info.boundingClientRect(function(data) { //data - 各种参数
			// 	self.mpHeight = data.height
			// }).exec()
			// // #endif
		},
		onLoad(option) {
			let that = this;
			if (this.is_diy) {
				if (uni.getStorageSync('FOOTER_BAR')) {
					uni.hideTabBar()
				}
				getNavigation().then(res => {
					this.newData = res.data
					if (this.newData.status && this.newData.status.status) {
						uni.hideTabBar()
					} else {
						uni.showTabBar()
					}
				})
			}
			
			// 获取积分
			getRankIntegral().then(res=> {
				that.integralRank = res.data;
			});
			

		}
	}
</script>

<style  lang="scss">
	.product-rank {
		padding: 10px 30rpx 0 0;
		display: flex;
		justify-content: start;
		.product-list {
			// border: 1px solid #FD502F;
			min-width: 20%;
			display: flex;
			flex-direction: column;
			align-items: left;
			.pli-border {
				background-color: unset !important;
			}
			.product-list-item {
				padding: 30rpx 30rpx 30rpx 30rpx;;
				background-color: #e8e8e8;
				display: flex;
				flex-direction: column;
				border-top: 1px solid #d6d6d6;
				.product-name {
					text-align: center;
					font-size: 25rpx;
				}
				.product-image {
					position: relative;
					width: 96rpx;
					height: 96rpx;
					border-radius: 10rpx;
				}
			}
		}
		.product-user-rank {
			// border: 1px solid #FD502F;
			padding-left: 20rpx;
			min-width: 79%;
			.pur-container {
				display: flex;
				align-items: center;
				justify-content: space-between;
				padding: 10px;
			}
			.nickname {
				width: 280rpx;
			}
		}
	}
	.itegral-rank-item {
		padding: 10px;
		display: flex;
		align-items: center;
		justify-content: space-around;
	}
	.avatar {
		position: relative;
		width: 96rpx;
		height: 96rpx;
		border-radius: 50%;
	}
	.tab-container {
		display: flex;
		justify-content: space-between;
		.tab-item {
			width: 50%;
			padding-top: 5px;
			padding-bottom: 5px;
			text-align: center;
		}
	}
		
	.new-users {
		display: flex;
		flex-direction: column;
		height: 100%;

		.sys-head {
			position: relative;
			width: 100%;
			// background: linear-gradient(90deg, $bg-star1 0%, $bg-end1 100%);

			.bg {
				position: absolute;
				left: 0;
				top: 0;
				width: 100%;
				height: 100%;
				background: var(--view-theme);
				background-size: 100% auto;
				background-position: left bottom;
			}

			.sys-title {
				z-index: 10;
				position: relative;
				height: 43px;
				text-align: center;
				line-height: 43px;
				font-size: 36rpx;
				color: #FFFFFF;
			}
		}
	}


	.page-footer {
		position: fixed;
		bottom: 0;
		z-index: 30;
		display: flex;
		align-items: center;
		justify-content: space-around;
		width: 100%;
		height: calc(98rpx+ constant(safe-area-inset-bottom)); ///兼容 IOS<11.2/
		height: calc(98rpx + env(safe-area-inset-bottom)); ///兼容 IOS>11.2/
		box-sizing: border-box;
		border-top: solid 1rpx #F3F3F3;
		background-color: #fff;
		box-shadow: 0px 0px 17rpx 1rpx rgba(206, 206, 206, 0.32);
		padding-bottom: constant(safe-area-inset-bottom); ///兼容 IOS<11.2/
		padding-bottom: env(safe-area-inset-bottom); ///兼容 IOS>11.2/

		.foot-item {
			display: flex;
			width: max-content;
			align-items: center;
			justify-content: center;
			flex-direction: column;
			position: relative;

			.count-num {
				position: absolute;
				display: flex;
				justify-content: center;
				align-items: center;
				width: 40rpx;
				height: 40rpx;
				top: 0rpx;
				right: -15rpx;
				color: #fff;
				font-size: 20rpx;
				background-color: #FD502F;
				border-radius: 50%;
				padding: 4rpx;
			}
		}

		.foot-item image {
			height: 50rpx;
			width: 50rpx;
			text-align: center;
			margin: 0 auto;
		}

		.foot-item .txt {
			font-size: 24rpx;


			&.active {}
		}
	}

</style>
