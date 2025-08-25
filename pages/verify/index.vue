<template>
	<view class="container">
		<view class="header-title">您即将进行身份认证</view>
		<view class="sub">为了您顺利地通过认证，请先确认设备与环境满足以下条件：</view>

		<view class="card">
			<image class="icon" src="/static/face-clear.svg" mode="widthFix" />
			<view class="content">
				<view class="header-row">
					<view class="num">1</view>
					<view class="card-title">五官无遮挡</view>
				</view>
				<view class="card-desc">请确保面部无口罩、眼镜等遮挡物，五官清晰可见</view>
			</view>
		</view>

		<view class="card">
			<image class="icon" src="/static/face-camera.svg" mode="widthFix" />
			<view class="content">
				<view class="header-row">
					<view class="num">2</view>
					<view class="card-title">正面对准手机</view>
				</view>
				<view class="card-desc">将手机正对面部，保持30-50厘米距离</view>
			</view>
		</view>

		<view class="card">
			<image class="icon" src="/static/phone-light.svg" mode="widthFix" />
			<view class="content">
				<view class="header-row">
					<view class="num">3</view>
					<view class="card-title">调节屏幕亮度</view>
				</view>
				<view class="card-desc">将手机屏幕亮度调至最大，以确保面部光照充足</view>
			</view>
		</view>

		<button class="primary" @click="start">开始认证</button>

		<view class="tip">
			<text class="dot">!</text>
			<text>如不满足以上条件，可能导致认证失败，请确保准备充分后再开始</text>
		</view>
	</view>
</template>

<script lang="ts" setup>
import { verifyInit, startLivingDetect } from '@/uni_modules/eshunda-liveness'
import { ref, onMounted } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

// 请前往云市场购买( https://market.aliyun.com/detail/cmapi00071585#sku=yuncode6558500001)试用套餐, 
// 购买完成后，进入管理控制台：https://marketnext.console.aliyun.com/bizlist，可以找到APPCODE 字段，复制到下面即可
// TIPS 修改这个为你的appcode
const APPCODE = '替换为你的appcode'

// 认证类型和用户信息
const authType = ref('liveness') // 默认活体检测
const userInfo = ref({
	name: '',
	idNumber: ''
})

onLoad((options: any) => {
	console.log(options)
	authType.value = options.type
	// 从实人认证页面传递的用户信息
	if (options.name) {
		userInfo.value.name = decodeURIComponent(options.name)
	}
	if (options.idNumber) {
		userInfo.value.idNumber = decodeURIComponent(options.idNumber)
	}
	console.log('认证类型:', authType.value)
	console.log('用户信息:', userInfo.value)
})

const goBack = () => {
	uni.navigateBack()
}

// 简单的 POST 封装
const post = (url: string, data: any) => new Promise<any>((resolve, reject) => {
	// 将对象转换为 x-www-form-urlencoded 格式
	const formData = Object.keys(data)
		.map(key => `${encodeURIComponent(key)}=${encodeURIComponent(data[key])}`)
		.join('&')
	
	uni.request({
		url,
		method: 'POST',
		data: formData,
		header: { 
			'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
			'Authorization': 'APPCODE ' + APPCODE,
			'X-Ca-Nonce': uuid()
		},
		success: (res) => {
			if(res.statusCode == 200 || res.statusCode == 599){
				resolve(res.data)
			}else{
				console.log(res)
				// 阿里云请求错误，一般错误状态码会在header 字段中
				console.error("请求发生异常:" + res.header['X-Ca-Error-Message'])
				// 如遇到
				reject
			}
		},
		fail: reject
	})
})

// 2) 生成 uuid 作为 bizId
function uuid() {
	return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, 
		function(c) {
			const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8)
			return v.toString(16)
		})
}

const startVerify = async (token : string) =>{
	// 3) 调起活体
	const ret = await startLivingDetect(token)
	const { code, data, msg } = ret
	if(code !== 'SUCCESS'){
		uni.navigateTo({
			url: `/pages/result/index?code=${encodeURIComponent(ret?.code||'')}&msg=${encodeURIComponent(ret?.msg||'')}&data=${encodeURIComponent(ret?.data||'')}&ok=0&type=${authType.value}`
		})
	}else{
		// 根据认证类型请求不同的验证接口
		let verifyUrl = "https://fake.market.alicloudapi.com/verify"
		let requestData: any = { token, authVerify: data }
		
		switch (authType.value) {
			case 'liveness':
				verifyUrl = "https://fake.market.alicloudapi.com/verify"
				break
			case 'real':
				verifyUrl = "https://esdrp.market.alicloudapi.com/verify"
				break
			case 'face':
				verifyUrl = "https://fake.market.alicloudapi.com/face/verify"
				break
		}
		
		// 请求服务器进行服务器活体校验
		const authResp: any = await post(verifyUrl, requestData)
		let ok = authResp?.data.passed === 'T'
		uni.navigateTo({
			url: `/pages/result/index?code=${encodeURIComponent(authResp?.code||'')}&msg=${encodeURIComponent(authResp?.msg||'')}&data=${encodeURIComponent(authResp?.data.resultCode||'')}&ok=${ok?1:0}&type=${authType.value}`
		})
	}
}

const start = async () => {
	try {
		// 根据认证类型设置不同的参数
		let livingType = 2 // 默认活体检测
		// 1) 端侧初始化，拿到初始化信息（initMsg）
		let { code, data, msg } = await verifyInit({ livingType })
		console.log('初始化数据:', data)
		if (code !== 'SUCCESS') {
			uni.showToast({ title: '初始化失败', icon: 'none' })
			console.error('初始化失败', msg)
			return
		}

		// 根据认证类型请求不同的商品和API地址
		let getTokenUrl = undefined
		let requestData = undefined
		if(authType.value === 'liveness'){
			getTokenUrl = 'https://fake.market.alicloudapi.com/getToken'
			requestData = {
				authInit: data,
			}
		}
		else if (authType.value === 'real') {
			getTokenUrl = 'https://esdrp.market.alicloudapi.com/getToken'
			requestData = {
				authInit: data,
				certName: userInfo.value.name,
				certNo: userInfo.value.idNumber
			}
		}

		const authResp: any = await post(getTokenUrl, requestData)
		console.log(authResp)
		const token = authResp?.data.token
		if(!token){
			uni.showToast({ title: '获取token失败', icon: 'none' })
			console.error('获取 token 失败: ', authResp)
			return
		}
		
		startVerify(token)
	} catch (e) {
		uni.showToast({ title: '调用失败', icon: 'none' })
		console.error('调用失败:', e)
	}
}
</script>

<style scoped>
.container { padding: 24rpx; }
.nav { display: flex; align-items: center; height: 88rpx; }
.back { font-size: 48rpx; padding-right: 16rpx; }
.nav-title { font-size: 32rpx; font-weight: 600; }
.header-title { margin-top: 16rpx; font-size: 40rpx; font-weight: 700; text-align: center; }
.sub { color: #666; margin: 16rpx 0 24rpx; line-height: 1.6; }
.card { display: flex; align-items: center; background: #fff; border-radius: 16rpx; padding: 24rpx; margin-bottom: 20rpx; box-shadow: 0 8rpx 24rpx rgba(0,0,0,.06); }
.icon { width: 72rpx; height: 72rpx; margin-right: 20rpx; }
.header-row { display: flex; align-items: center; gap: 16rpx; margin-bottom: 8rpx; }
.num { width: 36rpx; height: 36rpx; border-radius: 50%; background: #2f86ff; color: #fff; display: flex; align-items: center; justify-content: center; font-weight: 600; font-size: 24rpx; }
.content { flex: 1; }
.card-title { font-size: 32rpx; font-weight: 600; margin-bottom: 0; line-height: 36rpx; }
.card-desc { color: #666; line-height: 1.6; }
.primary { margin: 40rpx auto 24rpx; background: #2f6cff; color: #fff; border: none; padding: 16rpx 32rpx; border-radius: 12rpx; font-size: 28rpx; width: 75%; display: block; }
.tip { display: flex; align-items: flex-start; gap: 12rpx; color: #f56c6c; background: #fff7f7; padding: 20rpx; border-radius: 12rpx; }
.dot { width: 32rpx; height: 32rpx; border-radius: 50%; background: #f56c6c; color: #fff; text-align: center; line-height: 32rpx; font-weight: 700; }
</style>

