<template>
	<view class="container">
		<view class="header-title">请选择您想体验的产品</view>
		
		<view class="card">
			<view class="card-header">
				<image class="icon" src="/static/face-camera.svg" mode="widthFix" />
				<view class="card-title">活体检测</view>
			</view>
			<view class="card-desc">验证用户是否为真实活体本人,甄别欺诈行为,保障用户的利益。</view>
			<button class="card-btn" @click="goVerifyPage('liveness')">开始体验</button>
		</view>

		<view class="card">
			<view class="card-header">
				<image class="icon" src="/static/face-clear.svg" mode="widthFix" />
				<view class="card-title">实人认证</view>
			</view>
			<view class="card-desc">通过实时采集的人像信息和公安库身份证信息进行比对,确保用户身份的真实性。</view>
			<button class="card-btn" @click="goVerifyPage('real')">开始体验</button>
		</view>


	</view>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { init } from '@/uni_modules/eshunda-liveness'
init()

const goVerifyPage = (type: string) => {
	if (type === 'real') {
		uni.navigateTo({ url: '/pages/real-auth/index' })
	} else {
		uni.navigateTo({ url: `/pages/verify/index?type=${type}` })
	}
}
</script>

<style scoped>
.container { padding: 24rpx; }
.header-title { margin-top: 16rpx; font-size: 40rpx; font-weight: 700; text-align: center; margin-bottom: 40rpx; }
.card { background: #fff; border-radius: 16rpx; padding: 24rpx; margin-bottom: 20rpx; box-shadow: 0 8rpx 24rpx rgba(0,0,0,.06); }
.card-header { display: flex; align-items: center; margin-bottom: 16rpx; }
.icon { width: 48rpx; height: 48rpx; margin-right: 16rpx; }
.card-title { font-size: 32rpx; font-weight: 600; line-height: 36rpx; }
.card-desc { color: #666; line-height: 1.6; margin-bottom: 20rpx; font-size: 28rpx; }
.card-btn { background: #2f6cff; color: #fff; border: none; padding: 16rpx 32rpx; border-radius: 12rpx; font-size: 28rpx; width: 80%; display: block; margin: 0 auto; }
</style> 