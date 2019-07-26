<template>
	<view class="Module">
		<view class="ModuleWarp">
			<view class="ModuleItem" style="background: #DDA29F;" @click="GoMuduleList">
				<view class="IconWrap">
					<image src="../../static/images/list.png" class="inputIcon"></image>
				</view>
				<text>工时记录</text>
			</view>
			<!-- FFB47C 9BE3A2 -->
			<view v-if="userInfo.fempid != 0" class="ModuleItem"  style="background: #A5A9BC;" @click="GoMuduleInsert">
				<view class="IconWrap">
					<image src="../../static/images/edit.png" class="inputIcon"></image>
				</view>
				<text>新增工时</text>
			</view>
		</view>
	</view>
</template>

<script>
	import { mapState } from 'vuex';
	export default {
		data() {
			return {
			}
		},
		computed: {
			...mapState({
			  userInfo: state => state.userInfo
			})
		},
		methods: {
			GoMuduleList () {
				uni.navigateTo({
					url: '/pages/list/index' 
				})
			},
			GoMuduleInsert () {
				uni.navigateTo({
					url: '/pages/insert/index' 
				})
			}
		}
	}
</script>

<style>
	.Module{
		width: 100vw;
		height: 100vh;
		background: #FFFFFF;
		display: flex;
		justify-content: center;
	}
	.ModuleWarp{
		width: calc(100% - 40upx);
		padding: 20upx;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	.ModuleItem{
		width: 345upx;
		height: 345upx;
		display: flex;
		flex-direction: column;
	}
	.ModuleItem text{
		margin-top: 60upx;
		margin-left: 20upx;
		font-size: 42upx;
		color: #FFFFFF;
	}
	.IconWrap{
		width: 120upx;
		height: 120upx;
		margin-top: 20upx;
		margin-left: 20upx;
		background: #FFFFFF;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.IconWrap image{
		width: 80upx;
		height: 80upx;
		background: #FFFFFF;
		display: block;
		margin: 20upx auto;
	}
	
</style>
