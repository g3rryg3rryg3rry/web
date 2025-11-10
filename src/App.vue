<template>
  <main class="p-6 max-w-md mx-auto text-center">
    <h1 class="text-3xl font-bold mb-6">抽優惠</h1>

    <!-- Not logged in -->
    <div v-if="!user">
      <input
        v-model="email"
        placeholder="Enter your email"
        class="border p-2 w-full mb-2"
      />
      <button
        @click="login"
        class="bg-blue-500 text-white p-2 rounded w-full"
      >
        Join Giveaway
      </button>
    </div>

    <!-- Logged in -->
    <div v-else>
      <h2 class="text-xl mb-4">您好 {{ user.email }}!</h2>
      <p>✅ 您已成功加入抽獎活動！</p>

      <!-- 🎁 New: giveaway draw section -->
      <div class="mt-6 p-4 border rounded bg-green-50">
        <h3 class="font-semibold mb-2">🎁 抽獎區</h3>
        <p class="mb-4">點擊下方按鈕抽出您的優惠！</p>

        <button
          @click="drawPrize"
          class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded"
        >
          開始抽獎
        </button>

        <p v-if="prize" class="mt-4 text-lg font-bold text-green-700">
          🎉 恭喜！您獲得：{{ prize }}
        </p>
      </div>

      <button
        @click="logout"
        class="mt-6 bg-gray-500 text-white p-2 rounded"
      >
        Logout
      </button>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from "vue"
import { createClient } from "@supabase/supabase-js"

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseKey = import.meta.env.VITE_SUPABASE_KEY
const supabase = createClient(supabaseUrl, supabaseKey)

const email = ref("")
const user = ref(null)
const prize = ref("")

async function login() {
  const { error } = await supabase.auth.signInWithOtp({
    email: email.value,
    options: {
      emailRedirectTo: "https://web-6rr3.onrender.com/",
    },
  })
  if (error) alert(error.message)
  else alert("Magic link sent! Check your email.")
}

async function logout() {
  await supabase.auth.signOut()
  user.value = null
  prize.value = ""
}

function drawPrize() {
  const prizes = ["5% OFF", "10% OFF", "免運優惠", "再接再厲 😅"]
  prize.value = prizes[Math.floor(Math.random() * prizes.length)]
}

onMounted(async () => {
  const { data } = await supabase.auth.getUser()
  if (data.user) {
    user.value = data.user
    await supabase.from("giveaway_entries").insert([{ email: data.user.email }])
  }

  supabase.auth.onAuthStateChange(async (_event, session) => {
    if (session?.user) {
      user.value = session.user
      await supabase.from("giveaway_entries").insert([{ email: session.user.email }])
    } else {
      user.value = null
    }
  })
})
</script>

<style>
body {
  font-family: sans-serif;
}
</style>
