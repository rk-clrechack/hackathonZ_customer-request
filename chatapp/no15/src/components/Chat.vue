<script setup>
import { inject, ref, reactive, onMounted } from "vue"
import io from "socket.io-client"

// #region constant
const LIMIT_TIME = 60000
// #endregion

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const socket = io()
// #endregion

// #region reactive variable
const chatContent = ref("")
const chatList = reactive([])
const userList = reactive([{userName: userName.value, isAwayFromKeyboard: false}])
const counter = ref(0)
const timer = ref(null)
// #endregion

// #region lifecycle
onMounted(() => {
  registSocketEvent()
  startTimer()
})
// #endregion

// #region browser event handler
// 投稿メッセージをサーバに送信する
const onPublish = () => {
  socket.emit("publishEvent", `${userName.value}さん：${chatContent.value}`)
  // 入力欄を初期化
  chatContent.value = ""
}

// 退室メッセージをサーバに送信する
const onExit = () => {
  const message = `${userName.value}さんが退室しました。`
  socket.emit("exitEvent", { userName: userName.value, message })
}

// メモを画面上に表示する
const onMemo = () => {
  // メモの内容を表示
  chatList.push(`${userName.value}さんのメモ：${chatContent.value}`)
  // 入力欄を初期化
  chatContent.value = ""
}

// 離席イベントをサーバに送信する
const onChangeStatus = () => {
  userList[0].isAwayFromKeyboard = true
  socket.emit("AwayFromKeyboardEvent", userList[0])
}
// #endregion

// #region socket event handler
// サーバから受信した入室メッセージ画面上に表示する
const onReceiveEnter = (data) => {
  chatList.push(data.message)
  // 入室したユーザーをユーザーリストに追加
  userList.push({userName: data.userName, isAwayFromKeyboard: false})
}

// サーバから受信した退室メッセージを受け取り画面上に表示する
const onReceiveExit = (data) => {
  chatList.push(data.message)
  // 退室したユーザーをユーザーリストから削除
  const index = userList.findIndex(user => user.userName == data.userName)
  if(index !== -1) {
    userList.splice(index, 1)
  }
}

// サーバから受信した投稿メッセージを画面上に表示する
const onReceivePublish = (data) => {
  chatList.push(data)
}

// サーバから受信した離席イベント
const onAwayFromKeyboard = (data) => {
  const index = userList.findIndex(user => user.userName == data.userName)
  if(index !== -1) {
    userList[index].isAwayFromKeyboard = data.isAwayFromKeyboard
  }
}

// マウスやキーボード操作によってカウンターと状態をリセット
const onResetCounter = () => {
  if(userList[0].isAwayFromKeyboard) {
    userList[0].isAwayFromKeyboard = false
    socket.emit("AwayFromKeyboardEvent", userList[0])
  }
  clearTimeout(timer.value)
  counter.value = 0
  startTimer()
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

  // 離席イベントを受け取ったら実行
  socket.on("AwayFromKeyboardEvent", (data) => {
    onAwayFromKeyboard(data)
  })
}

// タイマーを開始
const startTimer = () => {
  timer.value = setTimeout(onChangeStatus, LIMIT_TIME)
}
// #endregion
</script>

<template>
  <div class="mx-auto my-5 px-4" @mousemove="onResetCounter" @keydown="onResetCounter">
    <h1 class="text-h3 font-weight-medium">Vue.js Chat チャットルーム</h1>
    <h2 class="mt-4">入室者一覧</h2>
    <ul>
      <li class="ml-6" :class="{'is-away-from-keyboard': user.isAwayFromKeyboard}" v-for="(user, index) in userList" :key="index">{{ user.userName }}</li>
    </ul>
    <div class="mt-10">
      <p>ログインユーザ：{{ userName }}さん</p>
      <v-textarea variant="outlined" v-model="chatContent" hide-details="false" placeholder="投稿文を入力してください" rows="4" class="mt-5 area"></v-textarea>
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

.area {
  width: 500px;
}

.item {
  display: block;
}

.is-away-from-keyboard {
  opacity: 0.5;
}
</style>