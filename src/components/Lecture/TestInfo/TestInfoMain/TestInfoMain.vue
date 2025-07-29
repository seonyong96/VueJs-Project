<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModalState } from '@/stores/modalState';
import TestInfoModal from '../TestInfoModal/TestInfoModal.vue';

const route = useRoute();
const testInfoList = ref([]);
const testInfoCount = ref(0);
const modalState = useModalState();
const detailLecId = ref(0);
const detailTestId = ref(0);

const testInfoSearch = (cPage = 1) => {
  const param = new URLSearchParams(route.query);
  param.append('currentPage', cPage);
  param.append('pageSize', 5);

  axios.post('/api/lecture/testInfoListBody.do', param).then((res) => {
    testInfoList.value = res.data.list;
    testInfoCount.value = res.data.count;
  });
};

watch(
  () => route.query,
  () => {
    testInfoSearch();
  },
);

onMounted(() => {
  testInfoSearch();
});

const testInfoDeatil = (lecId, testId) => {
  modalState.$patch({ isOpen: true });
  modalState.$patch({ type: 'testInfo' });
  detailLecId.value = lecId;
  detailTestId.value = testId;
};
</script>

<template>
  <div class="notice-main-container">
    <table class="notice-table">
      <thead class="notice-table-header">
        <tr>
          <th>강의</th>
          <th>강사</th>
          <th>강의실</th>
          <th>시험 시작일</th>
          <th>시험 종료일</th>
          <th>시험 등록일</th>
          <th>시험문제보기</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="testInfoCount > 0">
          <tr v-for="testInfo in testInfoList" :key="testInfo.testId" class="notice-table-row">
            <td
              class="font-bold hover:underline"
              @click="testInfoDeatil(testInfo.lecId, testInfo.testId)"
            >
              {{ testInfo.lecName }}
            </td>
            <td class="notice-cell">
              {{ testInfo.lecInstructorName }}
            </td>
            <td class="notice-cell">{{ testInfo.lecRoomName }}</td>
            <td class="notice-cell">{{ testInfo.testBeginDate }}</td>
            <td class="notice-cell">{{ testInfo.testEndDate }}</td>
            <td class="notice-cell">{{ testInfo.testRegDate.substr(0, 10) }}</td>
            <td class="notice-cell"><button class="button">시험문제보기</button></td>
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
      :total-items="testInfoCount"
      :items-per-page="5"
      :on-page-change="testInfoSearch"
    />
  </div>
  <testInfoModal
    v-if="modalState.isOpen && modalState.type === 'testInfo'"
    :detail-lec-id
    :detail-test-id
    @post-success="noticeSearch()"
    @un-mounted-modal="detailId = $event"
  />
</template>

<style scoped>
@import './styled.css';
</style>
