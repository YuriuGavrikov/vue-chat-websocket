<script setup>
import { ref } from "vue";

const messages = ref([]);
const value = ref("");
const socket = ref("");
const username = ref("");
const connected = ref(false);

function connect() {
   // socket.current = new WebSocket("ws://localhost:5000");
   socket.current = new WebSocket(
      "wss://inquisitive-awesome-lifter.glitch.me/"
   );

   socket.current.onopen = () => {
      connected.value = true;
      const message = {
         event: "connection",
         username: username.value,
         id: Date.now(),
      };
      socket.current.send(JSON.stringify(message));
   };
   socket.current.onmessage = (event) => {
      const message = JSON.parse(event.data);
      messages.value.unshift(message);
   };
   socket.current.onclose = () => {
      console.log("Socket закрыт");
   };
   socket.current.onerror = () => {
      console.log("Socket произошла ошибка");
   };
}

const sendMessage = async () => {
   const message = {
      username: username.value,
      message: value.value,
      id: Date.now(),
      event: "message",
   };
   socket.current.send(JSON.stringify(message));
   value.value = "";
};
</script>

<template>
   <div v-if="!connected">
      <div class="vstack gap-3 col-md-6 mx-auto">
         <div class="form-floating mb-3">
            <input
               v-model="username"
               @keypress.enter="connect"
               type="text"
               class="form-control"
               id="userName"
               placeholder="User name"
            />
            <label for="floatingInput">User name</label>
         </div>
         <button @click="connect" type="button" class="btn btn-outline-success">
            Send
         </button>
      </div>
   </div>

   <div v-else>
      <div class="vstack gap-3 col-md-6 mx-auto">
         <div>
            You: <span class="fw-bold text-success">{{ username }}</span>
         </div>
         <div class="form-floating">
            <textarea
               v-model="value"
               @keypress.enter="sendMessage"
               class="form-control"
               placeholder="Message"
               id="userMessage"
               style="min-height: 100px"
            ></textarea>
            <label for="floatingTextarea">Message</label>
         </div>
         <button
            @click="sendMessage"
            type="button"
            class="btn btn-outline-success"
         >
            Send
         </button>
         <div v-for="mess in messages" :key="mess.id" class="text-center">
            <div
               v-if="mess.event === 'connection'"
               class="card text-bg-success d-inline-block"
            >
               <div class="card-body">
                  Пользователь {{ mess.username }} подключился
               </div>
            </div>
            <div v-else class="card">
               <div class="card-body">
                  <span class="fw-bold text-success"
                     >{{ mess.username }}:
                  </span>
                  <span>{{ mess.message }}</span>
               </div>
            </div>
         </div>
      </div>
   </div>
</template>
