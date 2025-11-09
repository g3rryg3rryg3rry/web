<template>
  <main class="p-6 max-w-md mx-auto text-center">
    <h1 class="text-3xl font-bold mb-6">🎁 Vue Giveaway</h1>

    <div v-if="!user">
      <input v-model="email" placeholder="Enter your email" class="border p-2 w-full mb-2" />
      <button @click="login" class="bg-blue-500 text-white p-2 rounded w-full">Join Giveaway</button>
    </div>

    <div v-else>
      <h2 class="text-xl mb-4">Welcome, {{ user.email }}!</h2>
      <p>✅ You’ve joined the giveaway!</p>
      <button @click="logout" class="mt-4 bg-gray-500 text-white p-2 rounded">Logout</button>
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

async function login() {
  const { error } = await supabase.auth.signInWithOtp({
    email: email.value,
    options: {
      emailRedirectTo: window.location.origin,
    },
  })
  if (error) alert(error.message)
  else alert("Magic link sent! Check your email.")
}

async function logout() {
  await supabase.auth.signOut()
  user.value = null
}

onMounted(async () => {
  const { data } = await supabase.auth.getUser()
  if (data.user) {
    user.value = data.user
    // 加入抽獎資料表
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

