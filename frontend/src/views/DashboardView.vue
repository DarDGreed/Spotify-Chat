<template>
  <section class="app-main">
    <div class="min-h-screen flex flex-col bg-violet-400 text-white">
      <!-- Header -->
      <header class="bg-violet-700 py-4">
        <div class="container mx-auto px-4 flex justify-between items-center">
          <!-- Logo -->
          <div class="flex items-center">
            <h1 class="text-3xl font-bold cursor-pointer">TUF Vibin</h1>
          </div>
          <!-- User info and Logout button -->
          <div class="flex items-center">
            <p class="cursor-pointer">Welcome, <span class="text-sky-300 font-semibold">{{ username }}</span></p>
            <button @click="logout" class="bg-violet-800 hover:bg-violet-900 rounded-full px-4 py-2 ml-4">Logout</button>
          </div>
        </div>
      </header>

      <!-- Main Content -->
      <div class="flex flex-grow">
        <!-- Search and Track Selection -->
        <div class="flex-grow flex flex-col items-center mt-4">
          <!-- Search Input and Button -->
          <div class="flex items-center justify-center">
            <input v-model="searchQuery" type="text" placeholder="Search music . . ."
              class="text-center p-3 rounded-lg bg-violet-600 text-white placeholder-gray-400 focus:outline-none focus:bg-violet-700 border border-black">
            <button @click="search"
              class="ml-2 bg-violet-800 hover:bg-violet-900 text-white rounded-full px-6 py-3 font-semibold text-lg">Search</button>
          </div>

          <!-- Search Results Wrapper -->
          <div class="container mx-auto px-4 mt-8 overflow-y-auto h-96">
            <!-- Search Results -->
            <div v-for="track in tracks" :key="track.id" @click="selectTrack(track)"
              class="bg-violet-800 rounded-lg overflow-hidden shadow-lg p-2 mb-1 cursor-pointer flex items-center">
              <img :src="track.album.images[0].url" alt="Track Image" class="w-12 h-12 mr-4">
              <div>
                <div class="font-bold text-lg">{{ track.name }}</div>
                <p class="text-gray-400">{{ track.artists[0].name }}</p>
              </div>
              <button @click.stop="shareMessage(track.id)"
                class="ml-auto bg-violet-600 hover:bg-violet-700 text-white rounded-full px-2 py-1 text-sm">Copy Link</button>
            </div>
          </div>

          <!-- Selected Track -->
          <div v-if="selectedTrack" class="container mx-auto px-2 flex justify-center mt-2 bottom-center">
            <div class="w-full sm:w-3/4 p-2">
              <div class="bg-violet-800 rounded-lg overflow-hidden shadow-lg">
                <iframe :src="generateEmbedUrl(selectedTrack.id)" width="100%" height="153" frameborder=""
                  allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                  loading="lazy"></iframe>
              </div>
            </div>
          </div>
        </div>

        <!-- Messages on the Right -->
        <div class="w-1/3 bg-violet-800 border-l border-gray-700 p-3 overflow-y-auto" style="height: 675px;">
          <MessageView />
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import Swal from 'sweetalert2';
import { useRouter } from 'vue-router';
import MessageView from './MessageView.vue';

const searchQuery = ref('');
const tracks = ref([]);
const selectedTrack = ref(null);
const isLoggedIn = ref(false);
const username = ref('');
const router = useRouter();
const message = ref([]);

const topSongs = [
  { url: "https://open.spotify.com/embed/track/29WxJqIfDRMo9isV07kbJP", bgColor: "bg-violet-900" },
];

const search = async () => {
  try {
    const response = await axios.get('http://localhost:3000/search', {
      params: {
        q: searchQuery.value
      }
    });
    tracks.value = response.data.tracks.items;
  } catch (error) {
    console.error('Error searching tracks:', error);
  }
};

const generateEmbedUrl = (trackId) => {
  return `https://open.spotify.com/embed/track/${trackId}`;
};

const selectTrack = (track) => {
  selectedTrack.value = track;
};

const shareMessage = (trackId) => {
  const embedUrl = generateEmbedUrl(trackId);
  const message = `Check out this track: ${embedUrl}`;

  const input = document.createElement('input');
  input.style.position = 'fixed';
  input.style.opacity = 0;
  input.value = message;
  document.body.appendChild(input);
  input.select();
  document.execCommand('copy');
  document.body.removeChild(input);

  alert('Link copied to clipboard!');
};

const logout = () => {
  console.log("Logged out!");
  localStorage.removeItem('token');
  router.push('/');
  Swal.fire({
    icon: 'success',
    title: 'Logged out successfully',
    position: 'top',
    showConfirmButton: false,
    timer: 1500,
    customClass: {
      popup: 'my-swal-popup',
      title: 'my-swal-title'
    }
  });
};

onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:3000/dashboard', {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`
      }
    });
    message.value = response.data.message;
    isLoggedIn.value = true;

    const token = localStorage.getItem('token');
    const decodedToken = JSON.parse(atob(token.split('.')[1]));
    if (decodedToken) {
      username.value = decodedToken.username;
    }
  } catch (error) {
    console.error('Error fetching dashboard message:', error);
  }
});
</script>



<style>
  .bottom-center {
    position: fixed;
    bottom: 0;
    left: 33%;
    transform: translateX(-50%);
  }
.my-swal-popup {
  width: 200px;
  /* Adjust width as needed */
  background-color: #4CAF50;
  /* Custom background color */
  color: white;
  /* Custom text color */
}

.my-swal-title {
  font-size: 16px;
  /* Adjust font size as needed */
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter,
.slide-leave-to {
  transform: translateX(100%);
}
</style>
