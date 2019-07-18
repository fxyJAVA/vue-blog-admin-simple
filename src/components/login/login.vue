<template>
	<section class="login">
		<transition enter-active-class="shake" appear leave-active-class="fadeOut">
			<div class="login-box animated">
				<div class="avatar">
					<img src="../../assets/images/logo.png" draggable="false">
				</div>
				<div class="common">
					<label class="iconfont" for="username">&#xe617;</label>
					<input
						type="text"
						maxlength="20"
						autocomplete="off"
						placeholder="Username"
						id="username"
						v-model.trim="formData.username"
						@keyup.enter="login">
				</div>
				<div class="common">
					<label class="iconfont" for="password">&#xe641;</label>
					<input
						type="password"
						maxlength="20"
						autocomplete="off"
						placeholder="Password"
						id="password"
						v-model.trim="formData.password"
						@keyup.enter="login">
				</div>
				<div class="btn" @click="login">登 录</div>
			</div>
		</transition>
	</section>
</template>


<script>
export default {
	name: 'login',
	data () {
		return {
			flag: true,
            formData: {
                username: 'admin',
                password: 'admin',
            }
		}
	},
	methods: {
		// 点击登录
		login () {
		    this.$axios.post('/login',this.formData).then(response => {
		        //获取token
                localStorage.setItem('HEIHEIA',response.data)
                this.$message({
                    message: '登录成功, 准备为您跳转...',
                    type: 'success'
                });
                setTimeout(() => {
                    this.$router.push('/home/user');
                }, 1666);
                return;
            }).catch(error => {
                console.log(error)
            })
			// if ( this.username == 'admin' && this.password == 'admin' )
			// {
			// }
			// 如果登录失败
			this.$message({
				message: '账号或密码错误，请重试',
				type: 'error',
				duration: 1333,
	        });
		}
	},
}
</script>
