<template>
	<view class="container">
		<!-- 进度指示器 -->
		<view class="progress">
			<view class="progress-step active">
				<view class="step-number">1</view>
				<view class="step-text">填写信息</view>
			</view>
			<view class="progress-line"></view>
			<view class="progress-step">
				<view class="step-number">2</view>
				<view class="step-text">刷脸认证</view>
			</view>
		</view>

		<!-- 标题和说明 -->
		<view class="header">
			<view class="title">您即将进行核身验证</view>
			<view class="subtitle">为了您顺利地通过认证,请填写真实的身份信息</view>
		</view>

		<!-- 表单 -->
		<view class="form">
			<view class="form-item">
				<view class="label">姓名</view>
				<input 
					class="input" 
					v-model="formData.name" 
					placeholder="请输入您的真实姓名"
					placeholder-class="placeholder"
				/>
			</view>
			
			<view class="form-item">
				<view class="label">身份证号码</view>
				<input 
					class="input" 
					v-model="formData.idNumber" 
					placeholder="请输入18位身份证号码"
					placeholder-class="placeholder"
					maxlength="18"
				/>
			</view>
		</view>

		<!-- 按钮 -->
		<button class="next-btn" @click="nextStep" :disabled="!isFormValid">下一步</button>
	</view>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'

const formData = ref({
	name: '',
	idNumber: ''
})

const isFormValid = computed(() => {
	return formData.value.name.trim() && formData.value.idNumber.length === 18
})

const nextStep = () => {
	if (!isFormValid.value) {
		uni.showToast({ title: '请填写完整信息', icon: 'none' })
		return
	}
	
	// 验证身份证号码格式
	const idRegex = /^[1-9]\d{5}(18|19|20)\d{2}((0[1-9])|(1[0-2]))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/
	if (!idRegex.test(formData.value.idNumber)) {
		uni.showToast({ title: '身份证号码格式不正确', icon: 'none' })
		return
	}
	
	// 跳转到刷脸认证页面
	uni.navigateTo({
		url: `/pages/verify/index?name=${encodeURIComponent(formData.value.name)}&idNumber=${encodeURIComponent(formData.value.idNumber)}&type=real`
	})
}
</script>

<style scoped>
.container { 
	padding: 24rpx; 
	background: #f5f5f5;
	min-height: 100vh;
}

/* 进度指示器 */
.progress {
	display: flex;
	align-items: center;
	justify-content: center;
	margin-bottom: 40rpx;
	padding: 20rpx 0;
}

.progress-step {
	display: flex;
	flex-direction: column;
	align-items: center;
}

.step-number {
	width: 48rpx;
	height: 48rpx;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 24rpx;
	font-weight: 600;
	margin-bottom: 8rpx;
}

.progress-step.active .step-number {
	background: #2f6cff;
	color: #fff;
}

.progress-step:not(.active) .step-number {
	background: #e0e0e0;
	color: #999;
}

.step-text {
	font-size: 24rpx;
}

.progress-step.active .step-text {
	color: #2f6cff;
}

.progress-step:not(.active) .step-text {
	color: #999;
}

.progress-line {
	width: 120rpx;
	height: 2rpx;
	background: #e0e0e0;
	margin: 0 20rpx;
	margin-bottom: 24rpx;
}

/* 标题区域 */
.header {
	text-align: center;
	margin-bottom: 40rpx;
}

.title {
	font-size: 36rpx;
	font-weight: 700;
	color: #333;
	margin-bottom: 16rpx;
}

.subtitle {
	font-size: 28rpx;
	color: #666;
	line-height: 1.5;
}

/* 表单 */
.form {
	background: #fff;
	border-radius: 16rpx;
	padding: 32rpx;
	margin-bottom: 40rpx;
}

.form-item {
	margin-bottom: 32rpx;
}

.form-item:last-child {
	margin-bottom: 0;
}

.label {
	font-size: 28rpx;
	color: #333;
	margin-bottom: 16rpx;
	font-weight: 500;
}

.input {
	width: 100%;
	height: 88rpx;
	background: #f8f8f8;
	border-radius: 12rpx;
	padding: 0 24rpx;
	font-size: 28rpx;
	color: #333;
	box-sizing: border-box;
}

.placeholder {
	color: #999;
}

/* 按钮 */
.next-btn {
	width: 75%;
	height: 88rpx;
	background: #2f6cff;
	color: #fff;
	border: none;
	border-radius: 12rpx;
	font-size: 32rpx;
	font-weight: 600;
	display: block;
	margin: 0 auto;
}

.next-btn:disabled {
	background: #ccc;
	color: #999;
}
</style>
