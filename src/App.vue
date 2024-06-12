<script setup>
import CheckedList from "./components/checkedList.vue";
import list from "../public/list.json";
</script>

<template>
  <div id="box">
    <CheckedList :list="list"></CheckedList>
  </div>
</template>

<style lang="scss" scoped>
#box {
  width: 40%;
  height: 100%;
  margin: 100px auto;
  background-color: #f4f4f4;
}
</style>
