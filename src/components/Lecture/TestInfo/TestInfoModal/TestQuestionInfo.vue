<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, ref } from 'vue';

const { detailLecId: lecId, detailTestId: testId } = defineProps({
  detailLecId: { type: Number, default: 0 },
  detailTestId: { type: Number, default: 0 },
});

const questionOptionInfoList = ref([]);
const questionInfoList = ref([]);
const questionAnswerInfoList = ref([]);

const modalState = useModalState();
const detail = ref({});
const sum = ref(0);

const testQuestionDetail = () => {
  const param = {
    lecId: lecId,
    testId: testId,
  };

  axios.post('/api/lecture/testQuestionInfoDetail.do', param).then((res) => {
    questionOptionInfoList.value = res.data.testQuestionOptionInfoDetail;
    questionInfoList.value = res.data.testQuestionInfoDetail;
    questionAnswerInfoList.value = res.data.testQuestionAnswerInfoDetail;

    totalSum();
    console.log(sum.value);
  });
};

const testQuestionSave = () => {
  console.log();
};

const getCorrectOptionId = (questionId) => {
  const answer = questionAnswerInfoList.value.find((item) => item.questionId === questionId);
  return answer;
};

const totalSum = () => {
  let totalSum = 0;
  for (let i = 0; i < questionInfoList.value.length; i++) {
    totalSum += questionInfoList.value[i].questionScore;
    sum.value = totalSum;
  }
  return sum.value;
};

onMounted(() => {
  testQuestionDetail();
});

const closeModal = () => {
  modalState.$patch({ isOpen: false, type: null });
};
</script>

<template>
  <div class="modal-overlay" @click="closeModal">
    <div class="modal-container" @click.stop>
      <div class="modal-header">
        <h2>시험 상세</h2>
        <button class="close-btn" @click="closeModal">×</button>
      </div>

      <div class="modal-content">
        <div class="form-section">
          <h3>문제 등록</h3>

          <div class="form-row">
            <div class="form-group">
              <label>최대 문제 수</label>
              <input v-model="questionInfoList.length" type="number" />
            </div>
            <div class="form-group">
              <label>총 만점</label>
              <input v-model="sum" type="number" />
            </div>
          </div>

          <div class="form-row">
            <div class="form-group full-width">
              <label>배점</label>
              <input v-modal="questionScore" type="text" />
            </div>
            <div class="form-group">
              <label>정답 보기</label>
              <select>
                <option value="1번">1번</option>
                <option value="2번">2번</option>
                <option value="3번">3번</option>
                <option value="4번">4번</option>
                <option value="5번">5번</option>
              </select>
            </div>
          </div>

          <div class="form-group full-width">
            <label>문제 내용</label>
            <textarea rows="3"></textarea>
          </div>

          <div class="form-group full-width">
            <label>보기 1</label>
            <textarea rows="2"></textarea>
          </div>

          <div class="form-group full-width">
            <label>보기 2</label>
            <textarea rows="2"></textarea>
          </div>

          <div class="form-group full-width">
            <label>보기 3</label>
            <textarea rows="2"></textarea>
          </div>

          <div class="form-group full-width">
            <label>보기 4</label>
            <textarea rows="2"></textarea>
          </div>

          <div class="form-group full-width">
            <label>보기 5</label>
            <textarea rows="2"></textarea>
          </div>

          <button class="add-question-btn">문제 추가</button>
        </div>

        <div class="question-list-section">
          <h3>문제 추가 목록</h3>
          <table class="question-table">
            <thead>
              <tr>
                <th>번호</th>
                <th>내용</th>
                <th>배점</th>
                <th>정답</th>
                <th>삭제</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="questionInfo in questionInfoList" :key="questionInfo.questionNumber">
                <td>{{ questionInfo.questionNumber }}</td>
                <td>{{ questionInfo.questionContent }}</td>
                <td>{{ questionInfo.questionScore }}</td>
                <td>{{ getCorrectOptionId(questionInfo.questionId).correctOptionId }}번</td>
                <td>
                  <button class="delete-btn">삭제</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div class="modal-footer">
        <button class="submit-btn" @click="testQuestionSave()">수정</button>
        <button class="cancel-btn" @click="closeModal()">취소</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import './MarketDetailModal.css';
</style>
