<template>
  <template v-if="user">
    <van-cell title="当前用户" :value="user?.username" />
    <van-cell title="修改信息" is-link to="/user/update" />
    <van-cell title="我创建的队伍" is-link to="/user/team/create" />
    <van-cell title="我加入的队伍" is-link to="/user/team/join" />
    <van-cell title="退出登录" is-link @click="showLogoutConfirm"/>
  </template>

</template>

<script setup lang="ts">
import {useRouter} from "vue-router";
import {onMounted, ref} from "vue";
import myAxios from "../plugins/myAxios";
import {Toast,Dialog} from "vant";
import {getCurrentUser} from "../services/user";

// const user = {
//   id: 1,
//   username: '鱼皮',
//   userAccount: 'dogYupi',
//   avatarUrl: 'https://636f-codenav-8grj8px727565176-1256524210.tcb.qcloud.la/img/logo.png',
//   gender: '男',
//   phone: '123112312',
//   email: '12345@qq.com',
//   planetCode: '1234',
//   createTime: new Date(),
// }

const user = ref();

const router = useRouter();

onMounted(async () => {
  user.value = await getCurrentUser();
})

const toEdit = (editKey: string, editName: string, currentValue: string) => {
  router.push({
    path: '/user/edit',
    query: {
      editKey,
      editName,
      currentValue,
    }
  })
}
const showLogoutConfirm=()=>{
  Dialog.confirm({
    title: '退出登录',
    message: '确定要退出登录吗？',
  }).then(() => {
    logout();
  }).catch(() => {
    // on cancel
  });
}
const logout = async () => {
  try {
    // 执行登出操作，调用退出接口
    await myAxios.post('/user/logout');
    // 清除本地存储的用户信息等操作
    // 例如：localStorage.removeItem('user');
    // 重定向到登录页面或者其他需要的操作
    // 例如：router.push('/login');
    router.push('/'); // 跳转到首页
  } catch (error) {
    console.error('登出失败', error);
    Toast.fail('登出失败');
  }
}
</script>

<style scoped>

</style>
