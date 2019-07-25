<template>
	<view class="Insert">
		<!-- <view class="TopInfo">
			<text>项目编号：{{checkno}}</text>
			<text>日期：{{dateTxt}}</text>
			<text>操作人员：{{userInfo.fempid}}</text>
		</view> -->
		<view class="ListColumn">
			<!-- <text style="width: 20%;">项目</text> -->
			<!-- <text style="width: 20%;">工序</text> -->
			<!-- <text style="width: 20%;">备注</text> -->
			<text style="width: 10%;">序号</text>
			<text style="width: 25%">日期</text>
			<text style="width: 35%;">新项目编号</text>
			<text style="width: 20%;">工时</text>
			<text style="width: 10%;" class="ColorWhite">删除</text>
		</view>
		<view class="ListBlock">
			<view class="ListMain">
				<view v-for="(item, idx) in listData" :key="idx" style="margin-bottom: 0upx;">
					<view class="ListItem">
						<text style="width: 10%;">{{idx + 1}}</text>
						<picker style="width: 25%;text-align: center;" mode="date" :value="item.FDate" :data-idx="idx" :start="startDate" :end="endDate" @change="bindDateChange">
							<view class="uni-input">{{item.FDate}}</view>
						</picker>
						<input-autocomplete class="unit-item__input" style="width: 35%;text-align: center;border-bottom: 0px solid #CCCCCC;" :value="item.FBase2" v-model="item.FBase2" placeholder="请输入"
						 highlightColor="#FF0000" :loadData="loadAutocompleteData" v-on:selectItem="selectItemD"></input-autocomplete>
						<!-- <input v-model="item.FBase2" placeholder="请输入" style="width: 35%;text-align: center;border-bottom: 0px solid #CCCCCC;"/> -->
						<input v-model="item.FDecimal" placeholder="请输入" style="width: 20%;text-align: center;border-bottom: 0px solid #CCCCCC;"/>
						<view style="width: 10%;" @click="deleteLine(idx)">
							<image style="width: 40upx;height: 40upx;display: block;margin: 5upx auto;" src="../../static/images/delete.png"></image>
						</view>
					</view>
				</view>
				<text class="EmptyData" v-if="listData.length == 0">暂无数据</text>
			</view>
		</view>
		<view class="AddLine">
			<image @click="AddLine" style="width: 40upx;height: 40upx;display: block;margin: 10upx auto;" src="../../static/images/add.png"></image>
		</view>
		<button class="SubmitBt" :disabled="ifNoWork"  @click="save">保 存</button>
	</view>
</template>

<script>
	import { mapState } from 'vuex';
	import inputAutocomplete from '@/components/input-autocomplete/input-autocomplete.vue';
	export default {
		components: {
			inputAutocomplete
		},
		data() {
			return {
				ifNoWork: false,
				checkno: 'XSHT002848',
				dateTxt: '2019-07-28',
				supplier: '张三',
				listData: [],
				testObj: {
					sname: '静态',
					dname: '动态'
				}
			}
		},
		onLoad: function(option) {
			let that = this;
		},
		computed: {
			...mapState({
			  urlPre: state => state.urlPre,
			  userInfo: state => state.userInfo
			}),
			startDate() {
				return this.getDate('start');
			},
			endDate() {
				return this.getDate('end');
			}
		},
		methods: {
			AddLine () {
				this.listData.push({
					FDate: this.getDate({format: true}),
					FEmp: this.userInfo.fempid,
					FBase2: '',
					FDecimal: ''
				})
			},
			deleteLine (idx) {
				this.listData.splice(idx, 1)
			},
			bindDateChange (e) {
				var index = e.target.dataset.idx
				this.listData[index].FDate = e.target.value
			},
			getDate(type) {
				const date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();

				if (type === 'start') {
					year = year - 60;
				} else if (type === 'end') {
					year = year + 2;
				}
				month = month > 9 ? month : '0' + month;;
				day = day > 9 ? day : '0' + day;
				return `${year}-${month}-${day}`;
			},
			save () {
				let obj = {
					items: this.listData
				}
				this.submit('t_BOS200000004', JSON.stringify(obj))
			},
			submit (SOAPAction, Data) {
				uni.showLoading({
					title: '加载中'
				})
				
				var tmpData = '<?xml version="1.0" encoding="utf-8"?>'
				tmpData += '<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"> '
				tmpData += '<soap:Body> '
				tmpData += '<t_BOS200000004 xmlns="http://tempuri.org/">'
				tmpData += "<FJSONMSG>" + Data + "</FJSONMSG>"
				tmpData += '</t_BOS200000004>'
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
						let Result = xmlDoc.getElementsByTagName('t_BOS200000004Response')[0].getElementsByTagName('t_BOS200000004Result')[0]
						let HtmlStr = $(Result).html()
						let Info = (JSON.parse(HtmlStr))[0]
						console.log(Info)
						uni.hideLoading()
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
			loadAutocompleteData(value) {
			// 	let url = 'https://www.apiopen.top/journalismApi';
			// 	return uni
			// 		.request({
			// 			url: url
			// 		})
			// 		.then(ret => {
			// 			var [error, res] = ret;
			// 			console.log(res);
			// 			let data = (((res || {}).data || {}).data || {}).toutiao || [];
			// 			if (data.length <= 0) {
			// 				return Promise.resolve(['没有数据...']);
			// 			}
			// 
			// 			let retData = [];
			// 			for (let it of data) {
			// 				// console.log(it);
			// 				retData.push({
			// 					//自定义数据对象必须要有text属性
			// 					text: it.title,
			// 					//其它字段根据业务需要添加
			// 					digest: it.digest
			// 				});
			// 			}
			// 			//console.log(Promise.resolve(retData));
			// 			return Promise.resolve(retData);
			// 		});
			// 
				return Promise.resolve(['汉字行', 'da tang', '三人行', '大马路', '8哥', '我是动态数据']);
			},
			//响应选择事件，接收选中的数据
			selectItemD(data) {
				//选择事件
				console.log('收到数据了:', data);
			},
			selectItemS(data) {
				//选择事件
				console.log('收到数据了:', data);
			},
			printLog() {
				console.log(this.testObj);
			}
		}
	}
</script>

<style>
	.Insert{
		width: 100vw;
		min-height: 100vh;
		background: #FFFFFF;
		display: flex;
		flex-direction: column;
	}
	.TopInfo{
		width: 100%;
		padding: 20upx 0;
		height: 100upx;
	}
	.TopInfo text{
		width: calc(50% - 20upx);
		padding-left: 20upx;
		text-align: left;
		float: left;
		color: #333333;
	}
	.ListColumn{
		width: 100%;
		height: 60upx;
		/* border-top: 1px dashed #6DA977; */
		border-bottom: 1px dashed #6DA977;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	.ListColumn text{
		text-align: center;
		color: #6DA977;
	}
	.ListBlock{
		width: 100%;
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
		min-height: 100upx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: 1px dashed #DDDDDD;
	}
	.ListItem text{
		text-align: center;
		font-size: 22upx;
	}
	.uni-input-placeholder{
		font-size: 22upx;
		text-align: center;
	}
	.AddLine{
		width: 60upx;
		height: 60upx;
		border-radius: 50%;
		text-align: center;
		color: #0e932e;
		line-height: 60upx;
		border: 1px solid #0e932e;
		margin: 50upx auto 120upx auto;
	}
	.SubmitBt{
		width: 100%;
		height: 80upx;
		position: fixed;
		left: 0;
		bottom: 0;
		border-radius: 0;
		text-align: center;
		color: #FFFFFF;
		line-height: 80upx;
		background: #e64340;
	}
	.uni-input{
		padding: 0 !important;
	}
	/* automatic */
	.content {
		text-align: center;
		height: 100%;
	}
	.unit-title {
		font-size: 30upx;
		/* font-style: oblique; */
		color: #fff;
		padding: 16upx 26upx 50upx 26upx;
		padding-bottom: 10upx;
		text-align: left;
	}
	
	.unit-wrapper {
		padding: 44upx 0;
		margin: 0 30upx;
		border-radius: 32upx;
		margin-bottom: 20upx;
		background-color: #fff;
		color: #000;
	}
	
	.unit-item {
		display: flex;
		justify-content: flex-end;
		padding-right: 30upx;
		padding-left: 30upx;
		margin-bottom: 30upx;
	}
	
	.unit-item__header {
		margin-top: 0;
		margin-bottom: 8upx;
		padding: 0upx 8upx;
		display: flex;
		justify-content: space-between;
	}
	
	.unit-item__label {
		padding-top: 2px;
		text-align: right;
		font-size: 28upx;
		margin: 8upx 0 4upx 16upx;
		min-width: 188upx;
		width: 240upx;
	}
	
	.unit-item__input {
		text-align: left;
		width: 100%;
		font-size: 28upx;
		margin: 4upx 16upx 0 4upx;
		border: 2upx solid #eaeaea;
		border-radius: 12upx;
		min-height: 52.5upx;
		line-height: 52.5upx;
	}
</style>
