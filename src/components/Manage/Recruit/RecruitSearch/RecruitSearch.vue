<script setup>
import router from '@/router';
import { onMounted, ref } from 'vue';

const studentName = ref('');
const searchStDate = ref('');
const searchEdDate = ref('');

const handlerSearch = () => {
  const query = [];

  !studentName.value || query.push(`studentName=${studentName.value}`);
  !searchStDate.value || query.push(`startDate=${searchStDate.value}`);
  !searchEdDate.value || query.push(`endDate=${searchEdDate.value}`);

  const queryString = query.length > 0 ? `?${query.join('&')}` : '';

  router.push(queryString);
};

onMounted(() => {
  window.location.search && router.replace(window.localStorage.pathname);
});
</script>

<template>
  <div class="notice-container">
    <div class="input-box">
      제목: <input v-model.lazy="studentName" />
      <input v-model="searchStDate" type="date" />
      <input v-model="searchEdDate" type="date" />
      <button @click="handlerSearch">검색</button>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
