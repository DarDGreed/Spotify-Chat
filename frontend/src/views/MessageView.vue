<template>
  <div class="flex flex-col bg-violet-900 text-white">
    <div class="flex flex-col bg-violet-500 shadow-xl rounded-lg overflow-hidden border border-violet-400">
      <header class="bg-violet-900 py-4 border border-stone-500">
        <div class="flex items-center justify-between">
          <RouterLink class="text-2xl font-extrabold cursor-pointer font-sans text-white ml-6" to="/dashboard">
            <span class="text-white">Public Chat</span></RouterLink>
          <div class="flex items-center">
          </div>
        </div>
      </header>
      <div class="flex flex-col p-4 overflow-auto" style="height: 500px;">
        <div v-for="msg in messages" :key="msg.id" class="flex w-full mt-2">
          <div v-if="msg.username === username" class="flex items-end ml-auto">
            <div class="bg-violet-600 text-white p-3 rounded-l-lg rounded-br-lg">
              <p class="text-sm text-white">{{ msg.text }}</p>
              <iframe v-if="msg.embedUrl" :src="msg.embedUrl" class="w-full" height="250" frameborder="0"
                allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                loading="lazy"></iframe>
            </div>
            <span class="text-sm text-white-800 leading-none ml-3 font-bold mb-6">You</span>
          </div>
          <div v-else class="flex items-end">
            <div class="bg-gray-300 p-3 rounded-r-lg rounded-bl-lg">
              <p class="text-sm text-black">{{ msg.text }}</p>
              <iframe v-if="msg.embedUrl" :src="msg.embedUrl" class="w-full" height="250" frameborder="0"
                allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                loading="lazy"></iframe>
            </div>
            <span v-if="msg.username !== username" class="text-sm text-white leading-none ml-3 font-bold mb-6">{{ msg.username }}</span>
          </div>
        </div>
      </div>
      <div class="bg-violet-900 p-4 flex items-center space-x-2 border border-stone-500">
        <input v-model="newMessage" placeholder="Type a message" @keyup.enter="sendMessage"
          class="flex-grow h-10 rounded px-3 text-sm rounded-full text-black" />
        <button @click="sendMessage"
          class="bg-violet-700 hover:bg-violet-900 focus:outline-none text-white py-2 px-4 rounded-xl border-2 border-blue-300 font-semibold">
          <span>send</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { io } from 'socket.io-client';
import { ref, onMounted } from 'vue';

export default {
  data() {
    return {
      socket: null,
      messages: [],
      newMessage: '',
      loggedInUsername: ''
    };
  },
  setup() {
    const username = ref('');
    const isLoggedIn = ref(false);

    const fetchDashboard = async () => {
      try {
        const response = await axios.get('http://localhost:3000/dashboard', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });
        isLoggedIn.value = true;
        // Extract username from JWT token payload
        const token = localStorage.getItem('token');
        if (token) {
          try {
            const decodedToken = JSON.parse(atob(token.split('.')[1]));
            username.value = decodedToken.username;
          } catch (e) {
            // Handle token decode error
          }
        }
      } catch (error) {
        // Error fetching dashboard message
      }
    };
    onMounted(fetchDashboard);

    return { username, isLoggedIn };
  },
  mounted() {
    this.socket = io('http://localhost:3000');

    // Unbind any existing event listeners before binding a new one
    this.socket.off('chat message');

    this.socket.on('chat message', (msg) => {
      this.messages.push(msg);
      this.saveMessage(msg);
    });

    this.fetchMessages();
  },

  methods: {
    // Client-side code
    sendMessage() {
      if (this.newMessage.trim()) {
        // Check if the message is a Spotify link
        const spotifyRegex = /https:\/\/open\.spotify\.com\/embed\/track\/(\w+)/;
        const matches = this.newMessage.match(spotifyRegex);

        let messageData = {
          text: this.newMessage,
          username: this.username
        };

        // If it's a Spotify link and the regex match is successful, generate the embed URL
        if (matches && matches[1]) {
          const trackId = matches[1];
          messageData.track_id = trackId; // Assign the track_id
          messageData.embedUrl = `https://open.spotify.com/embed/track/${trackId}`;
        }

        // Send the message with or without the embed URL
        this.socket.emit('chat message', messageData);
        this.newMessage = '';
      }
    },
    async fetchMessages() {
      try {
        const response = await axios.get('http://localhost:3000/api/messages');
        console.log("Fetched messages:", response.data);
        this.messages = response.data;
      } catch (error) {
        console.error("Error fetching messages:", error);
      }
    },
    saveMessage(msg) {
      axios.post('http://localhost:3000/api/messages', msg)
        .catch(error => {
          console.error("Error saving message:", error);
        });
    }
  },
  beforeDestroy() {
    this.socket.disconnect();
  }
};
</script>
