<template>
  <van-cell center title="匹配模式">
    <template #right-icon>
      <van-switch v-model="isMatchMode" size="24" />
    </template>
  </van-cell>
    <UserCardList :user-list="userList" :loading="loading" />
    <van-empty v-if="!userList || userList.length < 1" description="搜索结果为空" />
  <van-back-top class="van-back-top" bottom="15vh" />
  <div style="text-align: center;
                padding: 20px 20px;
                column-span: all;"
  >
    <div style="text-align: center; padding: 20px 20px; column-span: all;">
      <van-button v-if="!isMatchMode && userList.length > 0" type="primary" size="small" @click="loadClick">点击加载更多</van-button>
    </div>
    <van-button v-if="showBackToTop" class="back-to-top" icon="arrow-up" type="primary" @click="scrollToTop"></van-button>
  </div>
</template>

<script setup lang="ts">
import {markRaw, onMounted, onUnmounted, ref, watch, watchEffect} from "vue";
import myAxios from "../plugins/myAxios";
import {Toast} from "vant";
import UserCardList from "../components/UserCardList.vue";
import {UserType} from "../models/user";
import {useRouter} from "vue-router";

const userList = ref([]);
const isMatchMode = ref<boolean>(false);
const loading = ref<boolean>(false);
const showBackToTop = ref(false);
//分页加载
const pageNumPre=ref(1);
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);
});
const handleScroll = () => {
  showBackToTop.value = window.scrollY > 500; // 示例逻辑
};
/**
 * 加载
 */
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth', // 可选，为了平滑滚动效果
  });
};
/**
 * 下拉刷新
 */

onMounted(() => {
  window.addEventListener('scroll', handleScroll);
});
/**
 * 点击加载更多
 */
const loadClick = async () => {
  let userListData;
  //匹配模式，根据标签匹配用户
  if (isMatchMode.value){
    const num=10;
    Toast.loading({
      message: '加载中',
      forbidClick: true
    })
    userListData = await myAxios.get('/user/match', {
      params: {
        num,
      },
    })
        .then(function (response) {
          console.log('/user/match success', response);
          Toast.success('加载成功');
          return response?.data; //返回数据 ?.可选链操作符，避免数据为 null
        })
        .catch(function (error) {
          console.log('/user/match error', error);
          Toast.fail('加载失败');
        })
  }else {
    //普通模式
    Toast.loading({
      message: '加载中',
      forbidClick: true
    })
    userListData = await myAxios.get('/user/recommend', {
      params: {
        pageSize: 5,
        pageNum: pageNumPre.value++
      },
    })
        .then(function (response) {
          console.log('/user/recommend success', response);
          Toast.success('加载成功');
          return response?.data?.records; //返回数据 ?.可选链操作符，避免数据为 null
        })
        .catch(function (error) {
          console.log('/user/recommend error', error);
          Toast.fail('加载失败');
        })
  }
  if (userListData) {
    console.log("url= " + userListData.avatarUrl);
    userListData.forEach((user: UserType) => {
      if (user.tags) {
        user.tags = JSON.parse(user.tags);
      }
    })
    // userListData.avatarUrl = "data:image/*;base64," + userListData.avatarUrl;
    // userListData.value = userListData.value.concat(userListData.data?.records);
    userList.value = userList.value.concat(userListData);
    console.log("list== " + userList.value.length);
  }
}

/**
 * 加载数据
 */
const loadData = async () => {
  loading.value = true;
  let userListData;
  //匹配模式，根据标签匹配用户
  Toast.loading({
    message: '加载中',
    forbidClick: true
  })
  if (isMatchMode.value){
    const num = 10;
    userListData = await myAxios.get('/user/match', {
      params: {
        num,
      },
    })
        .then(function (response) {
          console.log('/user/match success', response);
          Toast.success('加载成功');
          return response?.data; //返回数据 ?.可选链操作符，避免数据为 null
        })
        .catch(function (error) {
          console.log('/user/match error', error);
          Toast.fail('加载失败');
        })
  }else {
    //普通模式
    Toast.loading({
      message: '加载中',
      forbidClick: true
    })
    userListData = await myAxios.get('/user/recommend', {
      params: {
        pageSize: 5,
        pageNum: pageNumPre.value++,
      },
    })
        .then(function (response) {
          console.log('/user/recommend success', response);
          Toast.success('加载成功')
          return response?.data?.records; //返回数据 ?.可选链操作符，避免数据为 null
        })
        .catch(function (error) {
          console.log('/user/recommend error', error);
          Toast.fail('加载失败');
        })
  }
  if (userListData) {
    userListData.forEach((user: UserType) => {
      if (user.tags) {
        user.tags = JSON.parse(user.tags);
      }
      // if (user.avatarUrl){
      //   user.avatarUrl = 'http://localhost:3000/img' + user.avatarUrl;
      // }
    })
    userList.value = userListData;
  }

  loading.value = false;
}

watch(isMatchMode, () => {
  // 每次切换匹配模式时重置页码并重新加载数据
  pageNumPre.value = 1; // 重置页码为1
  loadData();
}, { immediate: true });


</script>

<style scoped>
.back-to-top {
  position: fixed;
  right: 30px;
  bottom: 80px;
  z-index: 1000; /* 确保按钮在页面上的其他元素之上 */
}
</style>
