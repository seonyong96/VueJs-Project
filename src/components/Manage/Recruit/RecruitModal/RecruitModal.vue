<script setup lang="ts">
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onUnmounted, ref } from 'vue';

const { detailId: id } = defineProps({ detailId: { type: String, default: '' } });
const modalState = useModalState();
const empName = ref('');
const empJoinDate = ref('');
const emit = defineEmits(['postSuccess', 'unMountedModal']);

const recruitSave = () => {
  const param = new URLSearchParams();
  param.append('loginID', id);
  param.append('empName', empName.value);
  param.append('empJoinDate', empJoinDate.value);

  axios.post('/api/manage/RecruitSave.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('저장되었습니다.');
      modalState.$patch({ isOpen: false, type: null });
      emit('postSuccess');
    }
  });
};

onUnmounted(() => {
  emit('unMountedModal', 0);
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay">
      <form ref="formRef" class="modal-form modal-container">
        <table>
          <tbody>
            <tr>
              <th>업체명</th>
              <td>
                <input v-model="empName" />
              </td>
            </tr>
            <tr>
              <th>입사일</th>
              <td>
                <input v-model="empJoinDate" type="date" />
              </td>
            </tr>
          </tbody>
        </table>
        <div class="button-container">
          <button type="button" @click="recruitSave()">저장</button>
          <button type="button" @click="modalState.$patch({ isOpen: false, type: null })">
            취소
          </button>
        </div>
      </form>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
