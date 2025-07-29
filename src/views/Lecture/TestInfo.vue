<script setup lang="ts">
import ContentBox from '@/components/common/ContentBox.vue';
import TestInfoMain from '@/components/Lecture/TestInfo/TestInfoMain/TestInfoMain.vue';
import TestInfoSearch from '@/components/Lecture/TestInfo/TestInfoSearch/TestInfoSearch.vue';
</script>

<template>
  <ContentBox>시험 정보</ContentBox>
  <TestInfoSearch />
  <TestInfoMain />
</template>
