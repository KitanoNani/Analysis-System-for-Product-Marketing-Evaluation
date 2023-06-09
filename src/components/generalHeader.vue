<template>
    <section class="el-header" v-loading.fullscreen.lock="fullscreenLoading">
        <section class="headerLogo">
            <h2>Æ</h2>
            <h4>～Analysis System for Product Marketing Evaluation～</h4>
        </section>

        <el-button-group v-if="!isLogin">
            <el-button type="primary">
                <RouterLink to="/auth">
                    Login
                </RouterLink>
            </el-button>
            <el-button type="primary">
                <RouterLink to="/auth?mod=reg">
                    Register
                </RouterLink>
            </el-button>
        </el-button-group>

        <section v-if="isLogin" id="userBar">
            <h3>Welcome, {{ usernameSession }} </h3>
            <el-dropdown>
                <el-button type="primary">
                    Options<el-icon class="el-icon--right"><arrow-down /></el-icon>
                </el-button>
                <template #dropdown>
                    <el-dropdown-menu>
                        <el-dropdown-item @click="{ changeNameVisible = true; }">Change Username</el-dropdown-item>
                        <el-divider></el-divider>
                        <el-dropdown-item @click="{ dialogVisible = true; }">
                            Log Out
                        </el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
        </section>
    </section>

    <el-dialog v-model="dialogVisible" title="Confirmation" width="30%" :show-close="false">
        <span>Are you sure to log out?</span>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="dialogVisible = false">Cancel</el-button>
                <el-button type="primary" @click="logout();">
                    Confirm
                </el-button>
            </span>
        </template>
    </el-dialog>

    <el-dialog v-model="changeNameVisible" title="Change Username?" width="30%" :show-close="false">
        <el-form ref="newNameFormRef" :model="newNameForm" label-width="180px">
            <el-form-item label="New Username" prop="username" :rules="[
                { required: true, message: 'New username is required' }
            ]">
                <el-input v-model="newNameForm.username" type="text" autocomplete="off" />
            </el-form-item>
        </el-form>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="changeNameVisible = false">Cancel</el-button>
                <el-button type="primary" @click="updateUsername();">
                    Confirm
                </el-button>
            </span>
        </template>
    </el-dialog>
</template>

<script setup lang="ts">
// import { useMainStore } from '@/stores'
import { ref, reactive } from 'vue'
import { ElMessage } from 'element-plus'
import * as opt from '@/utils/optimize'
import * as auth from '@/api/auth'
import { useMainStore } from '@/stores/user';
import pinia from '@/stores/';

/** Pinia存储 */
const store = useMainStore(pinia);
const fullscreenLoading = ref<Boolean>(false);
// const usernameStored = useMainStore().username;
const usernameSession = sessionStorage.getItem('username');
const isLogin = ref<Boolean>(false)
if (usernameSession && store.loginStatus
) {
    isLogin.value = true;
}

const dialogVisible = ref(false)
const changeNameVisible = ref(false)

const newNameForm = reactive({
    username: '',
    email: sessionStorage.getItem('id')
})

const updateUsername = async () => {
    fullscreenLoading.value = true;
    setTimeout(async () => {
        opt.debounce(async () => {
            if (!newNameForm.username) {
                ElMessage({
                    showClose: true,
                    message: '请输入内容',
                    type: 'error',
                    duration: 2500
                })
                fullscreenLoading.value = false;
                return;
            }
            let res = await auth.changeUsernameApi(newNameForm.username);
            fullscreenLoading.value = false;
            res = opt.formalizeRes(res)
            console.log(res);
            if (!res) return;
            if (res['avail']) {
                ElMessage({
                    showClose: true,
                    message: res['msg'],
                    type: 'success',
                    duration: 2500
                })
                // 正常情况下应该localStorage存token，由于项目时间紧、该部分不是着重点以及与后端的沟通本处采用session
                sessionStorage.setItem('username', newNameForm.username)
                store.setUsername('newNameForm.username');
                location.reload();
            } else {
                ElMessage({
                    showClose: true,
                    message: res['msg'],
                    type: 'error',
                    duration: 2500
                })
            }
        }, 1500, true)
    })
}

const logout = async () => {
    if (dialogVisible.value) {
        dialogVisible.value = false;
    }
    fullscreenLoading.value = true;
    setTimeout(async () => {
        opt.debounce(async () => {
            let res = await auth.logoutApi();
            fullscreenLoading.value = false;
            res = opt.formalizeRes(res);
            console.log(res);
            if (!res) return;
            if (res['avail']) {
                ElMessage({
                    showClose: true,
                    message: res['msg'],
                    type: 'success',
                    duration: 2500
                })
                // 正常情况下应该localStorage存token，由于项目时间紧、该部分不是着重点以及与后端的沟通本处采用session
                sessionStorage.clear();
                store.setUsername('');
                store.setId('');
                store.setLoginStatus(false);
                location.href = '/home';
            } else {
                ElMessage({
                    showClose: true,
                    message: res['msg'],
                    type: 'error',
                    duration: 2500
                })
            }
        }, 1000, true);
    }, 1000)
}

</script>

<style scoped lang="scss">
@import '../assets/scss/header.module.scss';
</style>