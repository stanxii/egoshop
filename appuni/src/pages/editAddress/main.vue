<template>
	<view class="tui-addr-box">
		<form :report-submit="true" @submit="formSubmit">
			<tui-list-cell :hover="false" padding="0">
				<view class="tui-line-cell">
					<view class="tui-title">收货人</view>
					<input :value="info.name" placeholder-class="tui-phcolor" class="tui-input" name="name" placeholder="请输入收货人姓名" maxlength="15" type="text" />
				</view>
			</tui-list-cell>
			<tui-list-cell :hover="false" padding="0">
				<view class="tui-line-cell">
					<view class="tui-title">手机号码</view>
					<input :value="info.mobile" placeholder-class="tui-phcolor" class="tui-input" name="mobile" placeholder="请输入收货人手机号码" maxlength="11"
					 type="text" />
				</view>
			</tui-list-cell>
			<tui-list-cell :arrow="true" padding="0">
				<view class="tui-line-cell">
					<view class="tui-title"><text class="tui-title-city-text">所在城市</text></view>
					<picker mode="region" @change="bindRegionChange">
						<input :value="region" placeholder-class="tui-phcolor" disabled class="tui-input" name="region" placeholder="请选择城市" maxlength="50" type="text" />
					</picker>

				</view>
			</tui-list-cell>
			<tui-list-cell :hover="false" padding="0">
				<view class="tui-line-cell">
					<view class="tui-title">收货地址</view>
					<input :value="info.detail" placeholder-class="tui-phcolor" class="tui-input" name="detail" placeholder="请输入详细的收货地址" maxlength="50" type="text" />
				</view>
			</tui-list-cell>
			<tui-list-cell :hover="false" padding="0">
				<view class="tui-line-cell">
					<view class="tui-cell-title">地址类型</view>
					<view class="tui-addr-label">
<!--						<text v-for="(item,index) in addressTypeList" :key="index" class="tui-label-item" :class="{'tui-label-active':index==1}">{{item.name}}</text>-->
						<text v-for="(item,index) in addressTypeList" :key="index" class="tui-label-item" @click="bindRadio(item)" :class="[typeId===item.id?'tui-label-active':'']">{{item.name}}</text>
					</view>
				</view>
			</tui-list-cell>

			<!-- 默认地址 -->
			<tui-list-cell :hover="false" padding="0">
				<view class="tui-swipe-cell">
					<view>设为默认地址</view>
					<switch :checked="info.isDefault===1" class="tui-switch-small" name="isDefault"/>
				</view>
			</tui-list-cell>
			<!-- 保存地址 -->
			<view class="tui-addr-save">
				<button type="warn" height="88rpx" form-type="submit">保存收货地址</button>
			</view>
			<view class="tui-del" v-if="false">
				<tui-button type="gray" height="88rpx">删除收货地址</tui-button>
			</view>
		</form>
	</view>
</template>

<script>
	import tuiButton from "@/components/button/button"
	import tuiListView from "@/components/list-view/list-view"
	import tuiListCell from "@/components/list-cell/list-cell"
	import {api} from "../../../config/api"
	const util = require('../../utils/util.js')

	export default {
		components: {
			tuiButton,
			tuiListView,
			tuiListCell
		},
		data() {
			return {
				lists: ["公司", "家", "学校", "其他"],
				region: "",
				customItem: "全部",
				addressTypeList: [],
				typeId: 0, // address的类型id
				info: {
					name: "",
					mobile: "",
					isDefault: 0,
					region: "",
					detail: "",
				},
			}
		},
		onShow() {
			if (this.$root.$mp.query.id) {
				this.id = this.$root.$mp.query.id;
				this.getDetail();
			}
			this.getAddressType()
		},
		methods: {
			async formSubmit(e) {
				let name = e.detail.value.name;
				let mobile = e.detail.value.mobile;
				let region = e.detail.value.region;
				let detail = e.detail.value.detail;
				let isDefault = e.detail.value.isDefault;
				console.log("e.detail.value",e.detail.value)
				if (util.isNullOrEmpty(mobile)) {
					this.tui.toast('请输入手机号码');
					return
				} else if (!util.isMobile(mobile)) {
					this.tui.toast('请输入正确的手机号码');
					return
				} else if (util.isNullOrEmpty(region)) {
					this.tui.toast('请输入城市');
					return
				} else if (util.isNullOrEmpty(name)) {
					this.tui.toast('请输入收货人名称');
					return
				}else if (util.isNullOrEmpty(detail)) {
					this.tui.toast('请输入详细地址');
					return
				}
				let isDefaultInt = isDefault?1:0


				let resp = null
				if (this.id !== undefined && this.id > 0) {
					resp = await this.wxhttp.route(api.address.update,{
						id: parseInt(this.id),
						name: name,
						mobile: mobile,
						detail: detail,
						isDefault: isDefaultInt,
						region: region,
						typeId: this.typeId
					});
				}else {
					resp = await this.wxhttp.route(api.address.create,{
						name: name,
						mobile: mobile,
						detail: detail,
						isDefault: isDefaultInt,
						region: region,
						typeId: this.typeId
					});
				}
				if (resp.code === 0) {
					uni.showToast({
						title: "添加成功", //提示的内容,
						icon: "success", //图标,
						duration: 2000, //延迟时间,
						mask: true, //显示透明蒙层，防止触摸穿透,
						success: res => {
							uni.navigateBack({
								delta: 1 //返回的页面数，如果 delta 大于现有页面数，则返回到首页,
							});
						}
					});
				}

			},
			async getAddressType() {
				const resp = await this.wxhttp.route(api.address.typeList);
				if (resp.code === 0) {
					this.addressTypeList = resp.data
				}
			},
			async getDetail() {
				const resp = await this.wxhttp.route(api.address.info,{
					id:parseInt(this.id),
				});
				if (resp.code === 0) {
					this.info = resp.data
					this.typeId = resp.data.typeId
					this.region = resp.data.region
				}
			},
			bindRadio(item) {
				console.log("item",item)
				this.typeId = item.id
				console.log("item",this.typeId)
			},
			bindRegionChange(e) {
				var value = e.mp.detail.value;
				this.region = value[0] + " " + value[1] + " " + value[2];
				console.log("this.region",this.region)
			},
		}
	}
</script>

<style>
	.tui-addr-box {
		padding: 20rpx 0;
	}

	.tui-line-cell {
		width: 100%;
		padding: 24rpx 30rpx;
		box-sizing: border-box;
		display: flex;
		align-items: center;
	}

	.tui-title {
		width: 180rpx;
		font-size: 28rpx;
	}

	.tui-title-city-text {
		width: 180rpx;
		height: 40rpx;
		display: block;
		line-height: 46rpx;
	}

	.tui-input {
		width: 500rpx;
	}

	.tui-input-city {
		flex: 1;
		height: 40rpx;
		font-size: 28rpx;
		padding-right: 30rpx;
	}

	.tui-phcolor {
		color: #ccc;
		font-size: 28rpx;
	}
	.tui-cell-title{
		font-size: 28rpx;
	}
	.tui-addr-label {
		margin-left: 70rpx;
	}

	.tui-label-item {
		width: 76rpx;
		height: 40rpx;
		border: 1rpx solid rgb(136, 136, 136);
		border-radius: 6rpx;
		font-size: 26rpx;
		text-align: center;
		line-height: 40rpx;
		margin-right: 20rpx;
		display: inline-block;
		transform: scale(0.9);
	}
	.tui-label-active{
		background: #E41F19;
		border-color:#E41F19;
		color: #fff;
	}
	.tui-swipe-cell {
		width: 100%;
		display: flex;
		justify-content: space-between;
		align-items: center;
		background: #fff;
		padding: 10rpx 24rpx;
		border-radius: 6rpx;
		overflow: hidden;
		font-size: 28rpx;
	}

	.tui-switch-small {
		transform: scale(0.8);
		transform-origin: 100% center;
	}

	/* #ifndef H5 */
	.tui-switch-small .wx-switch-input {
		margin: 0 !important;
	}

	/* #endif */

	/* #ifdef H5 */
	>>>uni-switch .uni-switch-input {
		margin-right: 0 !important;
	}

	/* #endif */

	.tui-addr-save {
		padding: 24rpx;
		margin-top: 100rpx;
	}

	.tui-del {
		padding: 24rpx;
	}
</style>
