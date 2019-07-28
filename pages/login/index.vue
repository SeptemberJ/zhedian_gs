<template>
	<view class="LoginWrap">
		<view class="TopBlock">
			<image class="Logo" src="../../static/images/LogoWhite.png"></image>
			<text>旺策尔PDA</text>
		</view>
		<view class="BotBlock">
			<view class="InputWrap">
				<view class="InputItem">
					<text>用户名</text>
					<input class="uni-input" v-model="username" placeholder="请输入用户名" style="border-bottom: 1px solid #000000;"/>
				</view>
				<view class="InputItem">
					<text>密码</text>
					<input class="uni-input" v-model="password" password placeholder="请输入密码"/>
				</view>
				<view class="LoginBt" @click="Login">
					<text>登 陆</text>
					<image class="LoginArrow" src="../../static/images/arrow.png"></image>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import { mapState, mapActions } from 'vuex'
	export default {
		data() {
			return {
				username: '', // 丁广 administrator潘晨星
				password: '' // Zhedian@123
			}
		},
		computed: {
			...mapState({
			  urlPre: state => state.urlPre
			})
		},
		methods: {
			...mapActions([
			  'updateUserInfo'
			]),
			Login () {
				if (this.username == '') {
					uni.showToast({
					    image: '/static/images/attention.png',
					    title: '用户名不能为空!'
					})
					return false
				}
				if (this.password == '') {
					uni.showToast({
					    image: '/static/images/attention.png',
					    title: '密码不能为空!'
					})
					return false
				}
				this.loginRequest('ja_login')
			},
			loginRequest () {
				uni.showLoading({
					title: '加载中'
				})
				uni.request({
					url: this.urlPre + '/userLogin?fname='+ encodeURIComponent(this.username) + '&password=' + this.password,
					// url: this.urlPre + '/userLogin?fname=潘晨星&password=11',
					method: 'POST',
					success: (res) => {
						console.log(this.username)
						console.log(this.password)
						console.log(res.data)
						switch (res.data.code) {
							case 1:
								// 更新用户信息
								this.updateUserInfo({'fempid': res.data.memberInfo.FItemID})
								uni.redirectTo({
									url: '/pages/module/index'
								})
								uni.hideLoading()
								break
							  default:
								uni.hideLoading()
								uni.showToast({
								    image: '/static/images/attention.png',
								    title: res.data.message + '!'
								})
						}
					},
					fail: (err) => {
						console.log('request fail', err)
						uni.hideLoading()
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				})
			},
			loginRequest2 (SOAPAction) {
				uni.showLoading({
					title: '加载中'
				})
				
				var tmpData = '<?xml version="1.0" encoding="utf-8"?>'
				tmpData += '<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"> '
				tmpData += '<soap:Body> '
				tmpData += '<ja_login xmlns="http://tempuri.org/">'
				tmpData += "<fuserno>" + this.username + "</fuserno>"
				tmpData += "<fuserpsw>" + this.password + "</fuserpsw>"
				tmpData += '</ja_login>'
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
						console.log('xml------------')
						console.log(xml)
						let parser = new DOMParser()
						let xmlDoc = parser.parseFromString(xml, 'text/xml')
						
						// 提取数据
						let Result = xmlDoc.getElementsByTagName('ja_loginResponse')[0].getElementsByTagName('ja_loginResult')[0]
						let HtmlStr = $(Result).html()
						let result = JSON.parse(HtmlStr)
						// console.log(result)
						switch (result.status) {
							case '1':
								// 更新用户信息
								this.updateUserInfo({'fempid': result.fempid})
								uni.redirectTo({
									url: '/pages/module/index'
								})
								uni.hideLoading()
								break
							  case '0':
								uni.hideLoading()
								uni.showToast({
								    image: '/static/images/attention.png',
								    title: result.message + '!'
								})
								break
							  default:
								uni.hideLoading()
								uni.showToast({
								    image: '/static/images/attention.png',
								    title: '服务器繁忙!'
								})
						}
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
.LoginWrap{
	width: 100vw;
	height: 100vh;
	background: #6DA977;
	display: flex;
	flex-direction: column;
	overflow: hidden;
}
.TopBlock, .BotBlock{
	width: 100vw;
	display: flex;
	justify-content: center!important;
	align-items: center!important;
}
.TopBlock{
	height: 40vh;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
}
.TopBlock text{
	font-size: 72upx;
	color: #ffffff;
	margin-top: 20upx;
}
.TopBlock .Logo{
	width: 250upx;
	height: 250upx;
}
.BotBlock{
	height: 60vh;
}
.BotBlock .InputWrap{
	width: 600upx;
	height: 400upx;
	padding-top: 35upx;
	background: #FFFFFF;
	display: flex;
	flex-direction: column;
}
.InputItem{
	height: 150upx;
	padding: 20upx 55upx 0 55upx;
	overflow: hidden;
	display: flex;
	flex-direction: column;
}
.InputItem text{
	color: grey;
}
.LoginBt{
	height: 100upx;
	font-weight: bold;
	background: #BCE1C3;
	padding: 0 55upx;
	display: flex;
	justify-content: space-between;
	align-items: center;
}
.LoginArrow{
	width: 60upx;
	height: 60upx;
}
.uni-input{
	padding: 7upx 0 !important;
}
.uni-input-placeholder{
	color: #000000 !important;
}
	
</style>
