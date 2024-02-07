<script setup>
import { inject, ref, reactive, onMounted } from "vue"
import io from "socket.io-client"

// #region global state
const userId = inject("userId")
const userName = inject("userName")
// #endregion

// #region local variable
const socket = io()
// #endregion

// #region reactive variable
const chatContent = ref("")
const chatList = reactive([])
const userList = reactive([{id: null, name: "全員"}])
const selectedUserId = ref(null)
// #endregion

// #region lifecycle
onMounted(() => {
  registSocketEvent()
})
// #endregion

// #region browser event handler
// 投稿メッセージをサーバに送信する
const onPublish = () => {
  const message = `${userName.value}さん：${chatContent.value}`
  socket.emit("publishEvent", {userId: selectedUserId.value, message})
  // 入力欄を初期化
  chatContent.value = ""
}

// 退室メッセージをサーバに送信する
const onExit = () => {
  const message = `${userName.value}さんが退室しました。`
  socket.emit("exitEvent", {userId: userId.value, message})
}

// メモを画面上に表示する
const onMemo = () => {
  // メモの内容を表示
  chatList.push(`${userName.value}さんのメモ：${chatContent.value}`)
  // 入力欄を初期化
  chatContent.value = ""
}
// #endregion

// #region socket event handler
// サーバから受信した入室メッセージ画面上に表示する
const onReceiveEnter = (data) => {
  chatList.push(data.message)
  // 入室したユーザーをユーザーリストに追加
  userList.push({id: data.userId, name: data.userName})
}

// サーバから受信した退室メッセージを受け取り画面上に表示する
const onReceiveExit = (data) => {
  chatList.push(data.message)
  // 退室したユーザーをユーザーリストから削除
  const index = userList.findIndex(user => user.id == data.userId)
  if(index !== -1) {
    userList.splice(index, 1)
  }
}

// サーバから受信した投稿メッセージを画面上に表示する
const onReceivePublish = (data) => {
  if(data.userId === null) {
    chatList.push(data.message)
    return
  }
  if(data.userId === userId.value) {
    chatList.push(`(DM) ${data.message}`)
  }
}
// #endregion

// #region local methods
// イベント登録をまとめる
const registSocketEvent = () => {
  // 入室イベントを受け取ったら実行
  socket.on("enterEvent", (data) => {
    onReceiveEnter(data)
  })

  // 退室イベントを受け取ったら実行
  socket.on("exitEvent", (data) => {
    onReceiveExit(data)
  })

  // 投稿イベントを受け取ったら実行
  socket.on("publishEvent", (data) => {
    onReceivePublish(data)
  })
}
// #endregion

</script>

<template>
  <div class="mx-auto my-5 px-4">
    <h1 class="text-h3 font-weight-medium">Vue.js Chat チャットルーム</h1>
    <div class="mt-10">
      <p>ID: {{ userId }}</p>
      <p>ログインユーザ：{{ userName }}さん</p>
      <v-textarea variant="outlined" v-model="chatContent" hide-details="false" placeholder="投稿文を入力してください" rows="4" class="mt-5 area"></v-textarea>
      <div class="d-flex align-center mt-5">
        <v-select
          class="select flex-grow-0 mr-3"
          :items="userList"
          item-title="name"
          item-value="id"
          v-model="selectedUserId"
          variant="outlined"
        >
        </v-select>
        <span>へ送信</span>
      </div>
      <div class="mt-4">
        <v-btn color="primary" variant="flat" @click="onPublish">投稿</v-btn>
        <v-btn color="primary" variant="flat" class="ml-1" @click="onMemo">メモ</v-btn>
      </div>
      <div class="mt-5" v-if="chatList.length !== 0">
        <ul>
          <li class="item mt-4" v-for="(chat, i) in chatList" :key="i">{{ chat }}</li>
        </ul>
      </div>
    </div>
    <router-link to="/" class="link">
      <v-btn color="primary" variant="flat" class="mt-5 btn" @click="onExit">退室する</v-btn>
    </router-link>
  </div>
</template>

<style scoped>
.link {
  text-decoration: none;
}

.select {
  width: 120px;
}

.area {
  width: 500px;
}

.item {
  display: block;
}

:deep(.v-input__details) {
  display: none;
}
</style>