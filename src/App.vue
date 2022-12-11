<script setup>
import axios from 'axios'
import { ref } from 'vue'

const baseUrl = "https://chat.kuriyama.top/api"
const qrKey = "/Auth/qr/key"
const qrCreate = "/Auth/qr/create"
const qrCheck = "/Auth/qr/check"
const key = ref('')
const qrCode = ref('')
const qrCodeText = ref('')
const userinfo = ref('')
const token = ref('')
const checkResult = ref('')

let timer

const getKey = async () => {
  const { status, data } = await axios.get(baseUrl + qrKey)
  if (status === 200 && data.code === 200) {
    key.value = data.key
    return data.key
  } else {
    console.error('get key error:', data)
  }
}

const createQrCode = async () => {
  if (!key.value) {
    alert('key is empty')
    return
  }
  const { status, data } = await axios.post(baseUrl + qrCreate, {
    key: key.value,
    qrimg: true
  })
  if (status === 200) {
    timer && clearInterval(timer)
    timer = setInterval(check, 2000)
    qrCode.value = data.image
    qrCodeText.value = data.url
    return data
  } else {
    console.error('get create error:', data)
  }
}

const check = async () => {
  const { data } = await axios.get(baseUrl + qrCheck, {
    params: {
      key: key.value
    }
  })
  checkResult.value = data
  if (data.code === 803) {
    timer && clearInterval(timer)
    token.value = data.accessToken
  } else if (data.code === 802) {
    userinfo.value = data.user
  }
}

</script>

<template>
  <main>
    <div>
      <button @click="getKey">获取key</button>
      <button @click="createQrCode">创建二维码</button>
      <h1>key:{{ key }}</h1>
      <h1>text:{{ qrCodeText }}</h1>
      <h1>result:{{ checkResult }}</h1>
      <img style="width:300px" :src="qrCode" v-if="qrCode">
    </div>
    <div>
      <div>
        userinfo:{{ userinfo }}
      </div>
      <div>
        token:{{ token }}
      </div>
    </div>
  </main>
</template>
<style>
* {
  margin: 0;
  padding: 0;
}

#app {
  height: 100vh;
}

main {
  display: flex;
}

main div {
  width: 50vh;
  flex: 1
}
</style>
