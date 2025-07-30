<script setup>
import router from '@/router';
import { useModalState } from '@/stores/modalState';
import { onMounted, ref } from 'vue';

const searchTag = ref('lecName');
const searchTitle = ref('');
const searchStDate = ref('');
const searchEdDate = ref('');
const modalState = useModalState();

const handlerSearch = () => {
  const query = [];

  !searchTitle.value || query.push(`searchTag=${searchTag.value}`);
  !searchTitle.value || query.push(`searchTitle=${searchTitle.value}`);
  !searchStDate.value || query.push(`searchStDate=${searchStDate.value}`);
  !searchEdDate.value || query.push(`searchEdDate=${searchEdDate.value}`);

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
      <select v-model="searchTag">
        <option value="lecName">강의명</option>
        <option value="lecInstructorName">강사명</option>
        <option value="lecRoomName">강의실</option>
      </select>
      <input v-model.lazy="searchTitle" />
      <input v-model="searchStDate" type="date" />
      <input v-model="searchEdDate" type="date" />
      <button @click="handlerSearch">검색</button>
      <button @click="modalState.$patch({ isOpen: true, type: 'testInfoNew' })">신규</button>
    </div>
  </div>
</template>

<style scoped>
@import './testInfoSearchStyled.css';
</style>
