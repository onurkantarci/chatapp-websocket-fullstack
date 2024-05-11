<template>
  <div v-if="!joined" class="parent-container">
    <div class="name-container">
      <input type="text" class="user-name" v-model="currentUser" />
      <button class="join-button" @click="join">Join</button>
    </div>
  </div>
  <div v-if="joined">
    <div class="list-container">
      <div v-for="message in messages" :key="message.id">
        <b>
          {{ message.user }}
        </b>
        : {{ message.text }}
      </div>
    </div>
    <div class="text-input-container">
      <textarea v-model="text" class="text-message" @keyup.enter="sendMessage">
      </textarea>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import io from "socket.io-client";

interface Message {
  id: number;
  text: string;
  user: string;
}

const joined = ref<boolean>(false);
const currentUser = ref<string>("");
const text = ref<string>("");
const messages = ref<Message[]>([]);
let socketInstance: any;

const join = () => {
  joined.value = true;

  socketInstance = io("http://localhost:3000");
  socketInstance.on("message:received", (data: any) => {
    messages.value = messages.value.concat(data);
  });
};

const sendMessage = () => {
  addMessage();
  text.value = "";
};

const addMessage = () => {
  const message = {
    id: new Date().getTime(),
    text: text.value,
    user: currentUser.value,
  };
  messages.value = messages.value.concat(message);
  socketInstance.emit("message", message);
};
</script>

<style scoped>
.parent-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  position: fixed;
  padding-top: 150px;
}

.name-container {
  display: flex;
  flex-direction: column;
  width: 200px;
}

.user-name {
  height: 30px;
  font-size: 20px;
  padding: 5px;
  margin-bottom: 5px;
  text-align: center;
  font-weight: bold;
}

.join-button {
  height: 30px;
  font-size: 20px;
}

.text-input-container {
  height: 100vh;
}

.text-message {
  width: 100%;
  position: absolute;
  bottom: 0px;
  height: 70px;
  padding: 10px;
  box-sizing: border-box;
}
</style>
