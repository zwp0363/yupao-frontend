<script setup lang="ts">
import {ref, watchEffect} from 'vue';
import myAxios from "../plugins/myAxios.ts";
import {showToast} from "vant";
import UserCardList from "../components/UserCardList.vue";
import type {UserType} from "../models/user";

const isMatchMode = ref<boolean>(false);

const userList = ref([]);
const loading = ref(true);

//加载数据
const loadData = async () => {
  let userListData;
  loading.value = true;
  // 心动模式，根据标签匹配用户
  if (isMatchMode.value) {
    const num = 5;
    userListData = await myAxios.get('/user/match', { // await等待请求全部结束才等到数据
      params: {
        num,
      },
    })
        .then(function (response) {
          console.log('/user/match succeed', response);
          return response?.data;
        })
        .catch(function (error) {
          console.error('/user/match error', error);
          showToast('请求失败');
        });
  } else {
    // 普通模式，直接分页查询用户
    userListData = await myAxios.get('/user/recommend', { // await等待请求全部结束才等到数据
      params: {
        pageSize: 8,
        pageNum: 1,
      },
    })
        .then(function (response) {
          console.log('/user/recommend succeed', response);
          return response?.data?.records;
        })
        .catch(function (error) {
          console.error('/user/recommend error', error);
          showToast('请求失败');
        });
  }
  if (userListData) {
    userListData.forEach((user:UserType) => {
      if (user.tags) {
          user.tags = JSON.parse(user.tags); //从 JSON 字符串解析成 JavaScript 对象
      }
    })
    userList.value = userListData;
  }
  loading.value = false;
}

// 监听模式变化，重新加载数据
watchEffect(() => {
  loadData();
})

</script>

<template>
  <van-cell center title="心动模式">
    <template #right-icon>
      <van-switch v-model="isMatchMode" size="24" />
    </template>
  </van-cell>
  <user-card-list :user-list="userList" :loading="loading"/>
  <van-empty v-if="!userList || userList.length < 1" description="数据为空"/>
</template>

<style scoped>

</style>