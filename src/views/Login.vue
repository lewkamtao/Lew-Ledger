<script lang="ts" setup>
    import { ref, onMounted } from 'vue';
    import { useRouter } from 'vue-router';
    import axios from '@/axios/http';
    const router = useRouter();

    let form = ref({
        username: 'kamtao',
        password: 'kamtao'
    });

    const login = () => {
        axios
            .post({
                url: `/login`,
                data: {
                    username: form.value.username,
                    password: form.value.password
                }
            })
            .then((res: any) => {
                if (res.code == 200) {
                    LewMessage.success('登录成功');
                    localStorage.setItem('token', res.data);
                    router.push('/');
                }
            });
    };

    onMounted(() => {});
</script>
<template>
    <lew-flex class="login-wrapper">
        <div class="login-form">
            <lew-title :bold="700" style="margin-bottom: 20px">登录 </lew-title>
            <lew-form-item direction="y" title="账号">
                <lew-input v-model="form.username" show-count :max-length="30" />
            </lew-form-item>
            <lew-form-item style="margin-bottom: 30px" direction="y" title="密码">
                <lew-input v-model="form.password" type="password" show-password />
            </lew-form-item>
            <lew-flex x="end">
                <lew-button @click="login">立即登录</lew-button>
            </lew-flex>
        </div>
    </lew-flex>
</template>
<style lang="scss">
    .login-wrapper {
        width: 100%;
        height: 100vh;
        background-color: #eee;
    }
    .login-form {
        padding: 30px;
        box-sizing: border-box;
        width: 300px;
        background-color: #fff;
        border-radius: var(--lew-form-border-radius);
    }
</style>
