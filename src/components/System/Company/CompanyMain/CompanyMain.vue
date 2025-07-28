<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';
import CompanyModal from '../CopanyModal/CompanyModal.vue';

const route = useRoute();
const companyList = ref([]);
const companyCount = ref(0);
const modalState = useModalState();
const detailId = ref(0);

const companySearch = (cPage = 1) => {
  const param = new URLSearchParams(route.query);
  param.append('currentPage', cPage);
  param.append('pageSize', 5);

  axios.post('/api/system/companyListBody.do', param).then((res) => {
    companyList.value = res.data.list;
    companyCount.value = res.data.count;
  });
};

watch(
  () => route.query,
  () => {
    companySearch();
  },
);

onMounted(() => {
  companySearch();
});

const companyDetail = (id) => {
  modalState.$patch({ isOpen: true });
  modalState.$patch({ type: 'company' });
  detailId.value = id;

  console.log('modalState', modalState);
  console.log('id', id);
};
</script>

<template>
  <div class="notice-main-container">
    <table class="notice-table">
      <thead class="notice-table-header">
        <tr>
          <th>번호</th>
          <th>회사명</th>
          <th>대표명</th>
          <th>휴대전화</th>
          <th>주소</th>
          <th>이메일</th>
          <th>가입일자</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="companyCount > 0">
          <tr v-for="company in companyList" :key="company.companyId" class="notice-table-row">
            <td class="notice-cell">{{ company.companyId }}</td>
            <td
              class="notice-cell cursor-pointer hover:underline"
              @click="companyDetail(company.companyId)"
            >
              {{ company.companyName }}
            </td>
            <td class="notice-cell">{{ company.companyCeo }}</td>
            <td class="notice-cell">{{ company.companyHp }}</td>
            <td class="notice-cell">{{ company.companyLoc }}</td>
            <td class="notice-cell">{{ company.companyEmail }}</td>
            <td class="notice-cell">
              {{ new Date(company.companyRegDate).toISOString().slice(0, 10) }}
            </td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="4" class="notice-empty-row">일치하는 검색 결과가 없습니다</td>
          </tr>
        </template>
      </tbody>
    </table>
    <PageNavigation
      :total-items="companyCount"
      :items-per-page="5"
      :on-page-change="companySearch"
    />
  </div>
  <CompanyModal
    v-if="modalState.isOpen && modalState.type === 'company'"
    :detail-id
    @post-success="companySearch()"
    @un-mounted-modal="detailId = $event"
  />
</template>

<style>
@import './styled.css';
</style>
