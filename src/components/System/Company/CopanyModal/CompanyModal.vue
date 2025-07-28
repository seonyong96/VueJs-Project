<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, onUnmounted, ref } from 'vue';

const modalState = useModalState();
const detail = ref({});
const emit = defineEmits(['postSuccess', 'unMountedModal']);
const { detailId: id } = defineProps({ detailId: { type: Number, default: 0 } });
const regDate = ref('');

const zipcode = ref('');
const roadAddress = ref('');
const formRef = ref();

defineOptions({
  name: 'DaumPostcodeComponent',
});

const compnayDetail = () => {
  // if (!id) {
  //   zipcode.value = this.zipcode.value;
  //   roadAddress.value = this.roadAddress.value;

  //   return;
  // }

  const param = new URLSearchParams();
  param.append('companyId', id);

  axios.post('/api/system/companyDetail.do', param).then((res) => {
    detail.value = res.data.detailValue;
    regDate.value = new Date(detail.value.companyRegDate).toISOString().slice(0, 10);
    zipcode.value = detail.value.zipcode;
  });
};

const companySave = () => {
  const param = new URLSearchParams();
  param.append('companyName', detail.value.companyName);
  param.append('companyCeo', detail.value.companyCeo);
  param.append('companyHp', detail.value.companyHp);
  param.append('companyLoc', zipcode.value);
  param.append('companyEmail', detail.value.companyEmail);
  param.append('companyRegDate', regDate.value);
  param.append('companyDetailAddress', detail.value.detailAddress);
  param.append('detailName', roadAddress.value);

  axios.post('/api/system/companySave.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('저장되었습니다.');
      emit('postSuccess');
      modalState.$patch({ isOpen: false, type: null });
      window.location.reload();
    }
  });
};

const companyUpdate = () => {
  const param = new URLSearchParams();
  param.append('companyId', id);
  param.append('companyName', detail.value.companyName);
  param.append('companyCeo', detail.value.companyCeo);
  param.append('companyHp', detail.value.companyHp);
  param.append('companyLoc', zipcode.value);
  param.append('companyEmail', detail.value.companyEmail);
  param.append('companyRegDate', regDate.value);
  param.append('companyDetailAddress', detail.value.detailAddress);
  param.append('detailName', roadAddress.value);

  axios.post('/api/system/companyUpdate.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('수정되었습니다.');
      emit('postSuccess');
      modalState.$patch({ isOpen: false, type: null });
    }
  });
};

const companyDelete = () => {
  const param = new URLSearchParams();
  param.append('companyId', id);

  axios.post('/api/system/companyDelete.do', param).then((res) => {
    if (res.data.result === 'success') {
      alert('삭제되었습니다.');
      emit('postSuccess');
      modalState.$patch({ isOpen: false, type: null });
    }
  });
};

/* global daum */
const execDaumPostcode = () => {
  new daum.Postcode({
    oncomplete: (data) => {
      zipcode.value = data.zonecode;
      roadAddress.value = data.roadAddress;

      console.log('zipcode.value => ', zipcode.value);
      console.log('roadAddress.value => ', roadAddress.value);
    },
  }).open();
};

// watch([() => zipcode.value, () => roadAddress.value], () => {
//   refresh
// })

onUnmounted(() => {
  emit('unMountedModal', 0);
});

onMounted(() => {
  id && compnayDetail();
});
</script>

<template>
  <div class="modal-overlay" @click="modalState.$patch({ isOpen: false, type: null })">
    <div class="modal-container" @click.stop>
      <!-- 모달 헤더 -->
      <div class="modal-header">
        <h2 class="modal-title">회사 정보</h2>
        <button class="close-button" @click="modalState.$patch({ isOpen: false, type: null })">
          ×
        </button>
      </div>

      <!-- 모달 본문 -->
      <div class="modal-body">
        <form ref="formRef">
          <!-- 회사명 -->
          <div class="form-group">
            <label class="form-label required">회사명</label>
            <input v-model="detail.companyName" type="text" class="form-input" />
          </div>

          <!-- 대표명 -->
          <div class="form-group">
            <label class="form-label required">대표명</label>
            <input v-model="detail.companyCeo" type="text" class="form-input" />
          </div>

          <!-- 휴대전화 -->
          <div class="form-group">
            <label class="form-label required">휴대전화</label>
            <input v-model="detail.companyHp" type="tel" class="form-input" />
          </div>

          <!-- 우편번호 -->
          <div class="form-group">
            <label class="form-label required">우편번호</label>
            <div class="zipcode-group">
              <input v-model="zipcode" type="text" class="form-input zipcode-input" readonly />
              <button type="button" class="zipcode-button" @click="execDaumPostcode">
                주소 검색
              </button>
            </div>
          </div>

          <!-- 기본주소 -->
          <div class="form-group">
            <label class="form-label required">기본주소</label>
            <input v-if="!roadAddress" v-model="detail.companyLoc" type="text" class="form-input" />
            <input v-else v-model="roadAddress" type="text" class="form-input" />
            <!-- <input v-model="detail.companyLoc" type="text" class="form-input" />
            <input v-model="roadAddress" type="text" class="form-input" /> -->
          </div>

          <!-- 상세주소 -->
          <div class="form-group">
            <label class="form-label required">상세주소</label>
            <input v-model="detail.detailAddress" type="text" class="form-input" />
          </div>

          <!-- 이메일 -->
          <div class="form-group">
            <label class="form-label required">이메일</label>
            <input v-model="detail.companyEmail" type="email" class="form-input" />
          </div>

          <!-- 가입일자 -->
          <div class="form-group">
            <label class="form-label required">가입일자</label>
            <input v-model="regDate" type="date" class="form-input" />
          </div>
        </form>
      </div>

      <!-- 모달 푸터 -->
      <div class="modal-footer">
        <button type="button" class="btn btn-submit" @click="id ? companyUpdate() : companySave()">
          {{ id ? '수정' : '저장' }}
        </button>
        <button v-if="id > 0" type="button" class="btn btn-delete" @click="companyDelete()">
          삭제
        </button>
        <button
          type="button"
          class="btn btn-cancel"
          @click="modalState.$patch({ isOpen: false, type: null })"
        >
          취소
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import './styled.css';
</style>
