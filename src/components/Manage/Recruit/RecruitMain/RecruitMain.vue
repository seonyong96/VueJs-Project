<script setup>
import PageNavigation from '@/components/common/PageNavigation.vue';
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import RecruitModal from '../RecruitModal/RecruitModal.vue';

const route = useRoute();
const recruitList = ref([]);
const recruitCount = ref(0);
const detailId = ref('');

const detailList = ref([]);
const detailCount = ref(0);

const modalState = useModalState();

const recruitSearch = (cPage = 1) => {
  const param = new URLSearchParams(route.query);
  param.append('currentPage', cPage);
  param.append('pageSize', 5);

  axios.post('/api/manage/recruitListBody.do', param).then((res) => {
    recruitList.value = res.data.list;
    recruitCount.value = res.data.count;
  });
};

watch(
  () => route.query,
  () => {
    recruitSearch();
  },
);

onMounted(() => {
  recruitSearch();
});

const recruitDetail = (id, page = 1) => {
  detailId.value = id;

  const param = new URLSearchParams();
  param.append('loginID', id);
  param.append('currentPage', page);
  param.append('pageSize', 5);

  axios.post('/api/manage/RecruitDetailBody.do', param).then((res) => {
    detailList.value = res.data.list;
    detailCount.value = res.data.count;
  });
};

const retireStudent = (id) => {
  if (confirm('정말 퇴직하시겠습니까?')) {
    const param = new URLSearchParams();
    param.append('loginID', id);

    axios.post('/api/manage/RetireStudent.do', param).then((res) => {
      if (res.data.result === 'success') {
        alert('퇴직되었습니다.');
        recruitDetail(id);
      }
    });
  }
};
</script>

<template>
  <div class="notice-main-container">
    <table class="notice-table">
      <thead class="notice-table-header">
        <tr>
          <th>로그인 ID</th>
          <th>학생명</th>
          <th>입사일</th>
          <th>퇴사일</th>
          <th>업체명</th>
          <th>재직여부</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="recruitCount > 0">
          <tr v-for="recruit in recruitList" :key="recruit.recruitId" class="notice-table-row">
            <td class="notice-cell">{{ recruit.loginID }}</td>
            <td
              class="notice-cell cursor-pointer hover:underline"
              @click="recruitDetail(recruit.loginID)"
            >
              {{ recruit.studentName }}
            </td>
            <td class="notice-cell">
              {{ new Date(recruit.empJoinDate).toISOString().slice(0, 10) }}
            </td>
            <td class="notice-cell">
              {{
                recruit.studentsEmpStatus === 'Y'
                  ? ''
                  : new Date(recruit.empRetireDate).toISOString().slice(0, 10)
              }}
            </td>
            <td class="notice-cell">{{ recruit.empName }}</td>
            <td class="notice-cell">{{ recruit.studentsEmpStatus === 'Y' ? '취업' : '실업' }}</td>
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
      :total-items="recruitCount"
      :items-per-page="5"
      :on-page-change="recruitSearch"
    />
    <template v-if="detailCount > 0">
      <div class="notice-main-container">
        <button class="button" @click="modalState.$patch({ isOpen: true, type: 'recruit' })">
          신규
        </button>
        <table class="notice-table">
          <thead class="notice-table-header">
            <tr class="notice-table-row">
              <th>번호</th>
              <th>학생명</th>
              <th>입사일</th>
              <th>퇴사일</th>
              <th>업체명</th>
              <th>재직여부</th>
              <th>-</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="detail in detailList" :key="detail.empId" class="notice-table-row">
              <td class="notice-cell">{{ detail.empId }}</td>
              <td class="notice-cell">{{ detail.studentName }}</td>
              <td class="notice-cell">
                {{ new Date(detail.empJoinDate).toISOString().slice(0, 10) }}
              </td>
              <td class="notice-cell">
                {{
                  detail.empRetireDate === null
                    ? ''
                    : new Date(detail.empRetireDate).toISOString().slice(0, 10)
                }}
              </td>
              <td class="notice-cell">{{ detail.empName }}</td>
              <td class="notice-cell">{{ detail.empRetireDate === null ? '취업' : '퇴직' }}</td>
              <td>
                <button
                  class="button"
                  :disabled="detail.empRetireDate"
                  @click="retireStudent(detail.loginID)"
                >
                  퇴직
                </button>
              </td>
            </tr>
          </tbody>
        </table>
        <PageNavigation
          :total-items="detailCount"
          :items-per-page="5"
          :on-page-change="(page) => recruitDetail(detailId, page)"
        />
      </div>
    </template>
    <RecruitModal
      v-if="modalState.isOpen && modalState.type === 'recruit'"
      :detail-id
      @post-success="recruitDetail(detailId)"
      @un-mounted-modal="detailId = $event"
    />
  </div>
</template>

<style scoped>
@import './styled.css';
</style>
