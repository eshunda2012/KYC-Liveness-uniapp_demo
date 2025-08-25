<template>
	<view class="wrap">
		<view class="card">
			<view class="title">认证结果</view>
			<view class="icon-wrap" :class="ok ? 'ok' : 'fail'">
				<text>{{ ok ? '✓' : 'X' }}</text>
			</view>
			<view class="list">
				<view class="row"><text class="label">状态码</text><text class="val">{{ code }}</text></view>
				<view class="row"><text class="label">消息</text><text class="val">{{ msg }}</text></view>
				<view class="row"><text class="label">结果码</text><text class="val">{{ data }}</text></view>
				<view class="row"><text class="label">验证结果</text><text class="val" :class="ok ? 'green' : 'red'">{{ ok ? '通过' : '失败' }}</text></view>
			</view>
			<button class="primary" @click="goHome">返回首页</button>
		</view>
	</view>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const code = ref('')
const msg = ref('')
const data = ref('')
const ok = ref(false)

onLoad((options: any) => {
	code.value = options?.code || ''
	msg.value = options?.msg || ''
	data.value = options?.data || ''
	ok.value = String(options?.ok) === '1'
})

const goHome = () => {
	uni.reLaunch({ url: '/pages/index/index' })
}
</script>

<style scoped>
.wrap { padding: 40rpx; display: flex; justify-content: center; }
.card { width: 650rpx; background: #fff; border-radius: 24rpx; padding: 40rpx; box-shadow: 0 16rpx 40rpx rgba(0,0,0,.08); }
.title { text-align: center; font-size: 36rpx; font-weight: 700; margin-bottom: 24rpx; }
.icon-wrap { width: 120rpx; height: 120rpx; border-radius: 60rpx; margin: 0 auto 24rpx; display: flex; align-items: center; justify-content: center; font-size: 48rpx; border: 6rpx solid; }
.icon-wrap.ok { color: #52c41a; border-color: #b7eb8f; background: #f6ffed; }
.icon-wrap.fail { color: #ff4d4f; border-color: #ffccc7; background: #fff2f0; }
.list { background: #f7f8fa; border-radius: 16rpx; padding: 16rpx; }
.row { display: flex; justify-content: space-between; padding: 18rpx 12rpx; border-bottom: 1px solid #eee; }
.row:last-child { border-bottom: 0; }
.label { color: #666; }
.val { color: #333; }
.green { color: #52c41a; }
.red { color: #ff4d4f; }
.primary { width: 280rpx; margin: 32rpx auto 0; display: block; background: #2f6cff; color: #fff; border: none; padding: 24rpx; border-radius: 16rpx; }
</style>

