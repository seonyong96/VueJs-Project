<script setup>
import router from '@/router';
import { useModalState } from '@/stores/modalState';
import { onMounted, ref } from 'vue';

const companyName = ref('');
const searchStDate = ref('');
const searchEdDate = ref('');
const modalState = useModalState();

const handlerSearch = () => {
  const query = [];

  !companyName.value || query.push(`title=${companyName.value}`);
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
      기업 이름: <input v-model.lazy="companyName" />
      <input v-model="searchStDate" type="date" />
      <input v-model="searchEdDate" type="date" />
      <button @click="handlerSearch">검색</button>
      <button @click="modalState.$patch({ isOpen: true, type: 'company' })">신규</button>
    </div>
  </div>
</template>

<style>
@import './styled.css';
</style>
