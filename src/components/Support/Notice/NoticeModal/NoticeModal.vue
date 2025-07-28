<script setup>
import { useModalState } from '@/stores/modalState';
import axios from 'axios';
import { onMounted, onUnmounted, ref } from 'vue';
const modalState = useModalState();
const formRef = ref();
const detail = ref({});

const emit = defineEmits(['postSuccess', 'unMountedModal']);
const { detailId: id } = defineProps({ detailId: { type: Number, default: 0 } });
const imageUrl = ref('');

const handlerInsert = () => {
  const formData = new FormData(formRef.value);

  console.log(formData);
  axios.post('/api/support/noticeFileSave.do', formData).then((res) => {
    if (res.data.result === 'success') {
      modalState.$patch({ isOpen: false });
      emit('postSuccess');
    }
  });
};

const handlerUpdate = () => {
  const formData = new FormData(formRef.value);
  formData.append('noticeId', id);

  axios.post('/api/support/noticeFileUpdate.do', formData).then((res) => {
    if (res.data.result === 'success') {
      alert('수정되었습니다.');
      modalState.$patch({ isOpen: false });
      emit('postSuccess');
    }
  });
};

const handlerDelelte = () => {
  const formData = new FormData(formRef.value);
  formData.append('noticeId', id);

  axios.post('/api/support/noticeFileDelete.do', formData).then((res) => {
    if (res.data.result === 'success') {
      modalState.$patch({ isOpen: false });
      alert('삭제되었습니다.');
      emit('postSuccess');
    }
  });
};

const searchDetail = () => {
  const param = new URLSearchParams();
  param.append('noticeId', id);

  axios.post('/api/support/noticeFileDetail.do', param).then((res) => {
    detail.value = res.data.detailValue;

    if (['jpg', 'gif', 'png'].includes(detail.value.fileExt)) {
      getFileImage();
    }
  });
};

// 상세조회 썸네일 구현
const getFileImage = () => {
  const param = new URLSearchParams();
  param.append('noticeId', id);

  axios.post('/api/support/noticeDownload.do', param, { responseType: 'blob' }).then((res) => {
    const url = window.URL.createObjectURL(new Blob([res.data]));
    imageUrl.value = url;
  });
};

// 미리보기 구현
const handlerFile = (e) => {
  // 현재 파일이 이미지 파일이면, 썸네일이 나타나게함 e <- @change 에 대한 정보가 들어감, e.target.files => 파일의 정보
  // console.log(e.target.files);

  // 선택된 파일의 확장자를 가져옴
  const fileInfo = e.target.files;
  const fileInfoSplit = fileInfo[0].name.split('.');
  // console.log(fileInfoSplit);
  const fileExtension = fileInfoSplit[1].toLowerCase();

  // 선택된 파일이 이미지파일 이라면 URL 생성
  if (['jpg', 'gif', 'png'].includes(fileExtension)) {
    // console.log(URL.createObjectURL(fileInfo[0]));
    imageUrl.value = URL.createObjectURL(fileInfo[0]);
  }
};

const downloadFile = () => {
  const param = new URLSearchParams();
  param.append('noticeId', id);

  axios.post('/api/support/noticeDownload.do', param, { responseType: 'blob' }).then((res) => {
    const url = window.URL.createObjectURL(new Blob([res.data]));
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', detail.value.fileName);
    document.body.appendChild(link);
    link.click();
    link.remove();
  });
};

onMounted(() => {
  id && searchDetail();
});

onUnmounted(() => {
  emit('unMountedModal', 0);
});
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay">
      <form ref="formRef" class="modal-form modal-container">
        <label> 제목 :<input v-model="detail.noticeTitle" type="text" name="fileTitle" /> </label>
        <label>
          내용 :<input v-model="detail.noticeContent" type="text" name="fileContent" />
        </label>
        파일 :
        <input id="fileInput" type="file" name="file" @change="handlerFile" />
        <label class="img-label" htmlFor="fileInput"> 파일 첨부하기 </label>
        <div>
          <div @click="downloadFile">
            <label>미리보기</label>
            <img class="preview-image" :src="imageUrl" />
          </div>
        </div>
        <div class="button-container">
          <button type="button" @click="!id ? handlerInsert() : handlerUpdate()">
            {{ !id > 0 ? '저장' : '수정' }}
          </button>
          <button v-if="id" type="button" @click="handlerDelelte">삭제</button>
          <button type="button" @click="modalState.$patch({ isOpen: false })">나가기</button>
        </div>
      </form>
    </div>
  </Teleport>
</template>

<style>
@import './styled.css';
</style>
