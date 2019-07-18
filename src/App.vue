<template>
    <div id="app">
        <keep-alive>
            <router-view></router-view>
        </keep-alive>
    </div>
</template>

<script>
export default {
    name: 'app',
    created () {
    	// 拦截器
    	var loading = null;
    	this.$axios.interceptors.request.use(config => {
    		loading = this.$loading({
    			text: '请求中...'
    		});
    		const token = localStorage.getItem('HEIHEIA');
    		if(token) {
    		    config.headers.common['YXFA'] = token
            }
    		return config;
    	}, e => {
			this.$message({
				message: '请求接口失败...',
				type: 'warning',
				duration: 1333,
	        });
    		return Promise.reject(e);
    	})

    	this.$axios.interceptors.response.use(res => {
    		loading.close();
    		let code = res.data.code
            if(code == 200) {
                return res.data
            }
            if(code == 711) {
                window.location.href = '/login'
            }
    		return Promise.reject(res.data);
    	}, e => {
    		loading.close();
    		return Promise.reject(e);
    	})
    },
}
</script>












