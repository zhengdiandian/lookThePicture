<template>
	<form  class="login-view">
		<view class="input-view">
			<view class="label-view">
				<text class="label">账号</text>
			</view>
			<input type="text" placeholder="请输入用户名" class="input" name="userName">
		</view>
		<view class="input-view">
			<view class="label-view">
				<text class="label">密码</text>
			</view>
			<input type="password" placeholder="请输入密码" class="input" name="PassWord">
		</view>
		<view class="button-view">
			<button class="login" houver-class="hover" form-type="submit">登入</button>
			<button class="register" houver-class="hover">免费注册</button>
		</view>
	</form>
</template>
<script>
export default {
	data() {
		return {};
	},
	methods: {},
	onLoad() {}
};
</script>
<style></style>
