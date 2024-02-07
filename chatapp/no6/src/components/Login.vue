<script setup>
import { inject, ref } from "vue"
import { useRouter } from "vue-router"
import io from "socket.io-client"

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const router = useRouter()
const socket = io()
// #endregion

// #region reactive variable
const inputUserName = ref("")
// #endregion

// #region browser event handler
// 入室メッセージをクライアントに送信する
const onEnter = () => {
  // ユーザー名が入力されているかチェック
  if(inputUserName.value.length === 0 ) {
    alert("ユーザー名を入力してください。")
    return
  }
  // 入室メッセージを送信
  socket.emit("enterEvent", `${inputUserName.value}さんが入室しました。`)
  // 全体で使用するnameに入力されたユーザー名を格納
  userName.value = inputUserName.value
  // チャット画面へ遷移
  router.push({ name: "chat" })
}
// #endregion
</script>

<template>
  <div class="mx-auto my-5 px-4">
    <h1 class="text-h3 font-weight-medium">Vue.js Chat サンプル</h1>
    <div class="mt-10">
      <p>ユーザー名</p>
      <v-text-field variant="outlined" density="compact" v-model="inputUserName" hide-details="false" class="user-name-text" />
    </div>
    <v-btn color="primary" variant="flat" class="mt-5" @click="onEnter">入室する</v-btn>
  </div>
</template>

<style scoped>
.user-name-text {
  width: 200px;
}
</style>
