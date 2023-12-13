<script setup>
import { nextTick, ref } from 'vue';
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: "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",
  dangerouslyAllowBrowser: true
})

const assistants = ref([
  { name: "ChatGPT", photo: "/images/logo.png", messages: [] },
  { name: "Yoda", photo: "/images/yoda.png", messages: [] },
  { name: "Sailor Moon", photo: "/images/sailor.png", messages: [] },
  { name: "Pikachu", photo: "/images/pikachu.png", messages: [] },
  { name: "Pirate", photo: "/images/pirate.png", messages: [] },
])
const assistant = ref(assistants.value[0])
const content = ref("")
const loading = ref(false)
const chat = ref(null)

const scroll = () => {
  chat.value.scrollTo({
    top: chat.value.scrollHeight,
    behavior: "smooth"
  })
}

const add = async e => {
  loading.value = true

  assistant.value.messages.push({
    role: "user",
    content: content.value
  })

  nextTick(scroll)

  content.value = ""

  const completion = await openai.chat.completions.create({
    model: "gpt-4-1106-preview",
    messages: [
      { role: "system", content: `Act as ${assistant.value.name}. Answer with one sentence.` },
      ...assistant.value.messages
    ],
  })

  assistant.value.messages.push(completion.choices[0].message)

  nextTick(scroll)

  loading.value = false
}
</script>

<template>
  <div class="bg-[#191B1D] h-screen grid place-items-center">
    <div class="max-w-6xl w-full shadow-xl grid grid-cols-4 h-[80vh]">
      <div class="bg-[#282C31] border-r-2 border-[#22262B]">
        <div class="p-4">
          <img src="/images/avatar.png" class="avatar">
        </div>
        <button v-for="a in assistants" :key="a.name" :disabled="loading" @click="assistant = a" :class="{ 'bg-[#33373C]': a == assistant }" class="p-4 flex items-center gap-4 w-full text-left">
          <img :src="a.photo" class="avatar">
          <div class="min-w-0">
            <h3 class="text-white text-sm font-semibold">{{ a.name }}</h3>
            <p class="text-white/50 text-sm truncate">{{ a.messages.length ? a.messages.at(-1).content : "No messages" }}</p>
          </div>
        </button>
      </div>
      <div class="col-span-3 flex flex-col overflow-auto">
        <header class="bg-[#282C31] p-4 flex items-center gap-4">
          <img :src="assistant.photo" class="avatar">
          <div>
            <h3 class="text-white text-sm font-semibold">{{ assistant.name }}</h3>
            <p class="text-white/50 text-sm">{{ assistant.messages.length }} messages</p>
          </div>
          <button @click="assistant.messages = []" class="ml-auto text-white/50">
            <i class="fa-solid fa-trash"></i>
          </button>
        </header>
        <div ref="chat" class="bg-[#22262B] flex-1 shadow-inner p-4 flex flex-col gap-4 overflow-auto">
          <div v-for="{ role, content } in assistant.messages" :class="{ 'flex-row-reverse': role == 'user' }" class="flex items-center gap-4">
            <img :src="role == 'user' ? '/images/avatar.png' : assistant.photo" class="avatar">
            <div :class="role == 'user' ? 'bg-[#BA1F4B]' : 'bg-[#3CA580]'" class="text-white p-4 text-sm rounded-xl max-w-[75%]">{{ content }}</div>
          </div>
        </div>
        <footer class="bg-[#282C31] p-4 relative">
          <div v-show="loading" class="absolute bottom-full mb-4 flex items-center gap-2">
            <div class="border-2 border-white w-4 aspect-square rounded-full border-t-transparent border-r-transparent animate-spin"></div>
            <p class="text-sm text-white/50"><span class="text-white font-semibold">{{ assistant.name }}</span> is typing...</p>
          </div>
          <form @submit.prevent="add" class="bg-[#33373C] p-4 rounded-xl flex gap-4">
            <input v-model.trim="content" required placeholder="Write a message" class="flex-1 bg-transparent outline-none text-white">
            <button :disabled="loading" class="text-white/50">
              <i class="fa-solid fa-paper-plane"></i>
            </button>
          </form>
        </footer>
      </div>
    </div>
  </div>
</template>

<style>
.avatar {
  @apply w-10 aspect-square rounded-full object-cover;
}
</style>