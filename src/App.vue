<script setup lang="ts">
import { Client, Account, Query, TablesDB } from 'appwrite'
import { ref } from 'vue'

const client = new Client()
client.setEndpoint('https://sgp.cloud.appwrite.io/v1').setProject('6993d4020013b6e4d07e')
const account = new Account(client)
const tablesDB = new TablesDB(client)

const enc = new TextEncoder()
const dec = new TextDecoder()
async function getKey(password, salt) {
  const baseKey = await crypto.subtle.importKey(
    'raw',
    enc.encode(password),
    { name: 'PBKDF2' },
    false,
    ['deriveKey'],
  )

  return crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt,
      iterations: 100000,
      hash: 'SHA-256',
    },
    baseKey,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt'],
  )
}
async function loadMsg(id: string, passcode: string) {
  try {
    const response = await tablesDB.listRows({
      databaseId: '6993d95900365e0bc682',
      tableId: '6993d99d002097cfa930',
      queries: [Query.equal('$id', id)],
    })

    const encrypted = response.rows[0]?.msg
    if (!encrypted) return ''

    msg.value = await decrypt(encrypted, passcode)
  } catch (error) {
    console.log(error)
    msg.value = 'Wrong passcode'
  }
}

function fromBase64(str: string) {
  return new Uint8Array([...atob(str)].map((c) => c.charCodeAt(0)))
}

async function decrypt(text, password) {
  const combined = fromBase64(text)
  const salt = combined.slice(0, 16)
  const iv = combined.slice(16, 28)
  const data = combined.slice(28)
  const key = await getKey(password, salt)
  const decrypted = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, data)

  return dec.decode(decrypted)
}

const ID = ref('')
const passcode = ref('')
const msg = ref('')
</script>

<template>
  <h1>UnEncode</h1>
  <div class="form">
    <input v-model="ID" placeholder="ID" />
    <input v-model="passcode" placeholder="Passcode" />
    <button @click="loadMsg(ID, passcode)">Load Data</button>
  </div>
  <div class="msg" v-html="msg"></div>
</template>

<style scoped>
h1 {
  text-align: center;
  margin-top: 1em;
}
.form,
.msg {
  width: 65ch;
  margin: auto;
  margin-top: 1em;
}
</style>
