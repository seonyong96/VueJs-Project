<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, onUnmounted, ref, watch } from 'vue';

const modalState = useModalState();
const { detailLecId: lecId, detailTestId: testId } = defineProps({
  detailLecId: { type: Number, default: 0 },
  detailTestId: { type: Number, default: 0 },
});
const detail = ref({});
const emit = defineEmits(['postSuccess', 'unMountedModal']);

const selectList = ref('');
const selectBox = ref('');
const selectedIns = ref('');
const selectedRoom = ref('');

const initializeState = () => {
  detail.value = {};
  selectBox.value = '';
  selectedIns.value = '';
  selectedRoom.value = '';
};

const testInfoSelectBox = () => {
  const param = new URLSearchParams();
  axios.post('/api/lecture/lectureSelectBoxList.do', param).then((res) => {
    selectList.value = res.data.lecSelectBoxList;
  });
};

const testInfoDetail = () => {
  const param = new URLSearchParams();
  param.append('lecId', lecId);
  param.append('testId', testId);

  axios.post('/api/lecture/testInfoDetail.do', param).then((res) => {
    detail.value = res.data.testInfoDetail;
    console.log(detail.value.lecName);
  });
};

const selectBoxHandler = () => {
  const selected = selectBox.value;

  selectedIns.value = selectList.value.find((item) => item.lecName === selected);
  selectedRoom.value = selectList.value.find((item) => item.lecName === selected);
};

const testInfoSave = () => {};

const testInfoUpdate = () => {};

watch(
  [() => lecId, () => testId],
  ([newLecId, newTestId], [oldLecId, oldTestId]) => {
    if (newLecId !== oldLecId || newTestId !== oldTestId) {
      initializeState();

      if (newLecId && newTestId) {
        testInfoDetail();
      }
    }
  },
  { immediate: false },
);

watch(selectBox, () => {
  selectBoxHandler();
});

onMounted(() => {
  if (lecId && testId) {
    testInfoDetail();
  }
  testInfoSelectBox();
});

onUnmounted(() => {
  emit('unMountedModal', 0);
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay">
      <div class="modal-container">
        <h2 class="modal-title">강의 상세</h2>
        <form ref="formRef" class="modal-form">
          <table>
            <tbody>
              <tr>
                <th>강의</th>
                <td>
                  <select v-if="lecId" :value="detail.lecName">
                    <option :disabled="true">{{ detail.lecName }}</option>
                  </select>
                  <select v-else v-model="selectBox">
                    <option>강의 선택</option>
                    <option v-for="select in selectList" :key="select.lecId">
                      {{ select.lecName }}
                    </option>
                  </select>
                </td>
              </tr>
              <tr>
                <th>강사</th>
                <td>{{ lecId ? detail.lecInstructorName : selectedIns.lecInstructorName }}</td>
              </tr>
              <tr>
                <th>강의실</th>
                <td>{{ lecId ? detail.lecRoomName : selectedIns.lecRoomName }}</td>
              </tr>
              <tr>
                <th>시험 시작일</th>
                <td>
                  <input v-model="detail.testBeginDate" type="datetime-local" />
                </td>
              </tr>
              <tr>
                <th>강의 종료일</th>
                <td>
                  <input v-model="detail.testEndDate" type="datetime-local" />
                </td>
              </tr>
            </tbody>
          </table>
          <div class="button-container">
            <button type="button" @click="!lecid ? testInfoSave() : testInfoUpdate()">
              {{ !lecId ? '저장' : '수정' }}
            </button>
            <button type="button" @click="modalState.$patch({ isOpen: false, type: null })">
              취소
            </button>
          </div>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
@import './styled.css';
</style>
