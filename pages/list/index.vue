<template>
	<view class="List">
		<view style="position: fixed;top: 0px;left: 0;z-index: 999;;">
			<view class="TopSearchWrap">
				<view class="TopSearchBar">
					<!-- <view class="InputItem" style="width: 30%;">
						<text>项目编号</text>
						<input class="uni-input" v-model="projectNo"/>
					</view> -->
					<view class="InputItem" style="width: 45%;">
						<text style="width: 100upx;">开始日期</text>
						<picker mode="date" :value="dateStart" @change="dateStartChange" style="width: calc(100% - 100upx);text-align: center;">
							<view class="uni-input">{{dateStart}}</view>
						</picker>
					</view>
					<view class="InputItem" style="width: 45%;">
						<text style="width: 100upx;">结束日期</text>
						<picker mode="date" :value="dateEnd" @change="dateEndChange" style="width: calc(100% - 100upx);text-align: center;">
							<view class="uni-input">{{dateEnd}}</view>
						</picker>
					</view>
					<!-- <view class="InputItem" style="width: 30%;">
						<text>状态</text>
						<picker style="width: 70%;" :value="status" :range="statusArray" @change="changeStatus">
							<view class="uni-input">{{statusArray[status]}}</view>
						</picker>
					</view> -->
					<view class="InputItem" style="width: 10%;">
						<image class="searchBt" style="" src="../../static/images/search.png" @click="search"></image>
					</view>
				</view>
				<view class="ListColumn">
					<text style="width: 150upx">日期</text>
					<text style="width: 100upx">职员</text>
					<text style="width: 150upx">部门</text>
					<text style="width: 100upx">项目编号</text>
					<text style="width: 100upx">工时</text>
					<text style="width: 150upx">备注</text>
				</view>
			</view>
		</view>
		<view class="ListBlock">
			<view class="ListMain">
				<view class="ListItem" v-for="(item, idx) in listData" :key="idx" @click="GoDetail(item)">
					<text style="width: 150upx">{{item['日期']}}</text>
					<text style="width: 100upx">{{item['职员']}}</text>
					<text style="width: 150upx">{{item['部门']}}</text>
					<text style="width: 100upx">{{item['项目编号']}}</text>
					<text style="width: 100upx">{{item['工时']}}</text>
					<text style="width: 150upx">{{item['备注']}}</text>
				</view>
			</view>
			<text class="EmptyData" v-if="listData.length == 0">暂无数据</text>
		</view>
		<view v-if="listData.length > 0" style="width: 100%;height:60upx;padding: 10upx 0;position: fixed;bottom: 0;left: 0;background: #FFFFFF;border-top: 1px solid #EEEEEE;">
			<uni-pagination :show-icon="true" title="标题文字" :pageSize="pageSize" :current="curPage" :total="total" @change="changePage"></uni-pagination>	
		</view>
	</view>
</template>
<script>
	import { mapState } from 'vuex';
	import uniPagination from '@/components/uni-pagination/uni-pagination.vue'
	export default {
		components: {
			uniPagination
		},
		data() {
			return {
				dateStart: '2019-07-01',
				dateEnd: '2019-07-25',
				// status: 3,
				curPage: 1,
				pageSize: 10,
				total: 0,
				// curContractNo: 'XSHT002848',
				// statusArray: ['待收料', '部分已审', '全部已审', '全部'],
				listData: []
			}
		},
		onLoad: function(option) {
			let that = this;
		},
		computed: {
			...mapState({
			  urlPre: state => state.urlPre,
			  userInfo: state => state.userInfo
			})
		},
		methods: {
			dateStartChange: function(e) {
				this.dateStart = e.target.value
			},
			dateEndChange: function(e) {
				this.dateEnd = e.target.value
			},
			// changeStatus (e) {
			// 	this.status = e.detail.value
			// },
			search () {
				this.curPage = 1
				this.getTotal('JA_LIST')
				this.getDataList('JA_LIST')
			},
			GoDetail () {
				
			},
			changePage (e) {
				this.curPage = e.current
				this.getDataList('JA_LIST')
			},
			getTotal (SOAPAction) {
				// uni.showLoading({
				// 	title: '加载中'
				// })
				
				var tmpData = '<?xml version="1.0" encoding="utf-8"?>'
				tmpData += '<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"> '
				tmpData += '<soap:Body> '
				tmpData += '<JA_LIST  xmlns="http://tempuri.org/">'
				tmpData += "<FSQL>exec [zz_searchBymonth_count] '" + this.userInfo.fempid + "','" + this.dateStart + "','" + this.dateEnd + "'</FSQL>"
				tmpData += '</JA_LIST >'
				tmpData += '</soap:Body>'
				tmpData += '</soap:Envelope>'
				
				uni.request({
					url: this.urlPre,
					method: 'POST',
					header: {
						'content-Type': 'text/xml; charset=utf-8',
						'SOAPAction': 'http://tempuri.org/' + SOAPAction
					 },
					dataType: 'json',
					data: tmpData,
					success: (res) => {
						let xml = res.data
						let parser = new DOMParser()
						let xmlDoc = parser.parseFromString(xml, 'text/xml')
						// 提取数据
						let Result = xmlDoc.getElementsByTagName('JA_LISTResponse')[0].getElementsByTagName('JA_LISTResult')[0]
						let HtmlStr = $(Result).html()
						let result = JSON.parse(HtmlStr)
						this.total = result[0].Column1
						console.log('总条数：' + result[0].Column1)
					},
					fail: (err) => {
						console.log('request fail', err)
						uni.hideLoading()
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					},
					complete: () => {
					}
				})
			},
			getDataList (SOAPAction) {
				// uni.showLoading({
				// 	title: '加载中'
				// })
				
				var tmpData = '<?xml version="1.0" encoding="utf-8"?>'
				tmpData += '<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"> '
				tmpData += '<soap:Body> '
				tmpData += '<JA_LIST  xmlns="http://tempuri.org/">'
				tmpData += "<FSQL>exec zz_searchBymonth '" + ((this.curPage - 1) * this.pageSize + 1) + "','" + this.curPage * this.pageSize + "','" + this.userInfo.fempid + "','" + this.dateStart + "','" + this.dateEnd + "'</FSQL>"
				tmpData += '</JA_LIST >'
				tmpData += '</soap:Body>'
				tmpData += '</soap:Envelope>'
				
				uni.request({
					url: this.urlPre,
					method: 'POST',
					header: {
						'content-Type': 'text/xml; charset=utf-8',
						'SOAPAction': 'http://tempuri.org/' + SOAPAction
					 },
					dataType: 'json',
					data: tmpData,
					success: (res) => {
						let xml = res.data
						let parser = new DOMParser()
						let xmlDoc = parser.parseFromString(xml, 'text/xml')
						// 提取数据
						let Result = xmlDoc.getElementsByTagName('JA_LISTResponse')[0].getElementsByTagName('JA_LISTResult')[0]
						let HtmlStr = $(Result).html()
						let result = JSON.parse(HtmlStr)
						this.listData = result
					},
					fail: (err) => {
						console.log('request fail', err)
						uni.hideLoading()
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					},
					complete: () => {
					}
				})
			}
		}
	}
</script>

<style>
	.List{
		width: 100vw;
		min-height: 100vh;
		background: #FFFFFF;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.TopSearchWrap{
		width: 100%;
		height: 120upx;
		display: flex;
		flex-direction: column;
	}
	.TopSearchBar{
		width: 100vw;
		height: 60upx;
		padding: 10upx 0;
		background: #6DA977;
		display: flex;
		justify-content: flex-start;
		align-items: center;
	}
	.InputItem{
		display: flex;
		align-items: center;
	}
	.InputItem text{
		padding: 0 10upx;
		color: #FFFFFF;
		font-size: 22upx;
	}
	.searchBt{
		width: 50upx;
		height: 50upx;
		margin: 0 auto;
	}
	.uni-input{
		padding: 0 10upx;
	}
	.ListColumn{
		width: 100%;
		height: 60upx;
		background: #FFFFFF;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: 1px dashed #ccc;
	}
	.ListColumn text{
		/* width: 16.66%; */
		text-align: center;
		/* padding: 0 15upx; */
		font-size: 22upx;
	}
	.ListBlock{
		width: 100%;
		background: #FFFFFF;
		position: relative;
		top: 120upx;
		display: flex;
		flex-direction: column;
		margin-bottom: 80upx;
	}
	.ListMain{
		width: 100%;
		display: flex;
		flex-direction: column;
	}
	.ListItem{
		width: 100%;
		padding: 10px 0;
		/* min-height: 60upx; */
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: 1px solid #EEEEEE;
	}
	.ListItem text{
		/* width: 16.66%; */
		text-align: center;
		/* padding: 0 15upx; */
		font-size: 22upx;
		color: #333333;
	}
</style>
