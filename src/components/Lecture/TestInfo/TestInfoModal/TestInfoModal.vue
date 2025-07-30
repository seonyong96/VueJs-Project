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

// const testBeginDate = ref('');
// const testEndDate = ref('');

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

    console.log('=================st', detail.value.testBeginDate);
    console.log('=================ed', detail.value.testEndDate);
  });
};

const selectBoxHandler = () => {
  const selected = selectBox.value;

  selectedIns.value = selectList.value.find((item) => item.lecName === selected);
  selectedRoom.value = selectList.value.find((item) => item.lecName === selected);
  console.log(selectList.value);
};

const testInfoSave = () => {
  const findLecId = selectList.value.find((item) => item.lecName === selectBox.value);

  const param = {
    lecId: findLecId.lecId,
    testBeginDate: detail.value.testBeginDate.replace('T', ' ') + ':00',
    testEndDate: detail.value.testEndDate.replace('T', ' ') + ':00',
    testType: 0,
  };

  axios.post('/api/lecture/testInfoSave.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('저장되었습니다.');
      modalState.$patch({ isOpne: false, type: null });
      emit('postSuccess');
    }
  });
};

const testInfoUpdate = () => {
  // const param = {
  //   lecId: detail.value.lecId,
  //   testBeginDate: detail.value.testBeginDate.replace('T', ' ') + ':00',
  //   testEndDate: detail.value.testEndDate.replace('T', ' ') + ':00',
  // };
  // const updateStdt = detail.value.testBeginDate;
  // const updateEddt = detail.value.testEndDate;

  const param = new URLSearchParams();
  param.append('lecId', detail.value.lecId);
  param.append('testId', detail.value.testId);
  param.append('testBeginDate', detail.value.testBeginDate.replace('T', ' ') + ':00');
  param.append('testEndDate', detail.value.testEndDate.replace('T', ' ') + ':00');

  console.log('-----------------st', detail.value.testBeginDate);
  console.log('-----------------ed', detail.value.testEndDate);
  axios.post('/api/lecture/testInfoUpdate.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('수정되었습니다.');
      closeModal();
      emit('postSuccess');
    }
  });
};

const closeModal = () => {
  modalState.$patch({ isOpen: false, type: null });
};

watch(selectBox, () => {
  selectBoxHandler();
});

onMounted(() => {
  testInfoSelectBox();
  if (modalState.type === 'testInfo') {
    testInfoDetail();
  }
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
                  <select v-if="modalState.type === 'testInfo'" :value="detail.lecName">
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
                <td>
                  {{
                    modalState.type === 'testInfo'
                      ? detail.lecInstructorName
                      : selectedIns.lecInstructorName
                  }}
                </td>
              </tr>
              <tr>
                <th>강의실</th>
                <td>
                  {{
                    modalState.type === 'testInfo' ? detail.lecRoomName : selectedIns.lecRoomName
                  }}
                </td>
              </tr>
              <tr>
                <th>시험 시작일</th>
                <td>
                  <input
                    v-model="detail.testBeginDate"
                    type="datetime-local"
                    name="testBeginDate"
                  />
                </td>
              </tr>
              <tr>
                <th>시험 종료일</th>
                <td>
                  <input v-model="detail.testEndDate" type="datetime-local" name="testEndDate" />
                </td>
              </tr>
            </tbody>
          </table>
          <div class="button-container">
            <button
              type="button"
              @click="modalState.type === 'testInfoNew' ? testInfoSave() : testInfoUpdate()"
            >
              {{ !lecId ? '저장' : '수정' }}
            </button>
            <button type="button" @click="closeModal()">취소</button>
          </div>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
@import './styled.css';
</style>
