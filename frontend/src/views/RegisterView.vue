<template>
  <header class="bg-violet-700 py-4">
      <div class="container mx-auto px-4 flex items-center justify-between">
        <!-- Logo -->
        <div class="flex items-center">
          <h1 class="text-white text-2xl font-extrabold">TUF Vibin</h1>
        </div>
        <!-- Navigation Links -->
        <nav class="space-x-4">
        </nav>
      </div>
    </header>
  <div
    class="min-h-screen flex items-center justify-center bg-violet-400 py-12 px-4 sm:px-6 lg:px-8"
  >
    <div class="max-w-md w-full">
  <div class="items-center bg-violet-600 p-8 py-12 rounded-md shadow-md space-y-8 w-96">
    <div class="flex items-center justify-center">
            <h1 class="text-white text-2xl font-semibold">TUF VIBIN</h1>
          </div>
    <h2 class="text-center text-3xl font-extrabold text-white">Vibe Now!</h2>
    <form @submit.prevent="register" class="space-y-6">
      <div class="rounded-md shadow-sm -space-y-px">
        <!-- Email Input -->
        <div>
          <label for="email" class="sr-only">Email</label>
          <input
            v-model="email"
            id="email"
            name="email"
            type="email"
            autocomplete="email"
            required
            class="mb-6 appearance-none rounded-full relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-full focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm"
            placeholder="Email"
          />
        </div>
        <!-- Username Input -->
        <div>
          <label for="username" class="sr-only">Username</label>
          <input
            v-model="username"
            id="username"
            name="username"
            type="text"
            autocomplete="username"
            required
            class="mb-6 appearance-none rounded-full relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm"
            placeholder="Username"
          />
        </div>
        <!-- Password Input -->
        <div>
          <label for="password" class="sr-only">Password</label>
          <input
            v-model="password"
            id="password"
            name="password"
            type="password"
            autocomplete="new-password"
            required
            class="appearance-none rounded-full relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm"
            placeholder="Password"
          />
        </div>
      </div>

      <!-- Register Button -->
      <div>
        <button
          type="submit"
          class="mb-4 w-full flex justify-center py-2 px-4 border border-transparent text-sm font-bold rounded-full text-white bg-violet-800 hover:bg-violet-900 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
        >
          <span class="flex items-center">
            REGISTER
          </span>
        </button>
        <h1 class="text-white font-semibold text-center">Already on Spotify? <span><RouterLink class="text-violet-800 hover:text-violet-900" to="/" >Login</RouterLink ></span></h1>
      </div>
    </form>
  </div>
</div>
</div>
</template>

<script setup>
import axios from "axios";
import { useRouter } from "vue-router";
import { ref } from "vue";
import Swal from 'sweetalert2';

const email = ref("");
const username = ref("");
const password = ref("");
const route = useRouter();

const register = async () => {
  try {
    const response = await axios.post("http://localhost:3000/register", {
      email: email.value,
      username: username.value,
      password: password.value,
    });

    console.log(response.data.message); // Output message from the backend
    // Redirect to login page after successful registration
    route.push("/");
    // Display toast alert
    Swal.fire({
      icon: 'success',
      title: 'Registered successfully',
      showConfirmButton: false,
      timer: 1500
    }); // Use SweetAlert for success message
  } catch (error) {
    console.error("Error registering:", error);
  }
};
</script>

