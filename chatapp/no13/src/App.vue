<script setup>
import { provide, ref } from "vue"

// #region reactive state
const userId = ref("")
const userName = ref("")
// #endregion

// #region global variable
provide("userId", userId)
provide("userName", userName)
// #endregion
</script>

<template>
  <router-view />
</template>

<style scoped>
</style>
