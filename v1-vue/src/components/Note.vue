<template>
  <div id="app" class="container">
    <h1>Collaborative Document</h1>
    <label for="text-area" class="label">Start writing your content:</label>
    <textarea
      id="text-area"
      v-model="text"
      placeholder="Start writing..."
      class="text-area"
    ></textarea>
    <button @click="updateText" class="update-button">Update Document</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import PieSocket from "piesocket-js";

const pieSocket = new PieSocket({
  clusterId: "free.blr2",
  apiKey: "B9UKgvptNTWrZxfCUTquFp7nKVsYqu2LtmBao5Jg",
});

let text = ref(localStorage.getItem("text") || "");
let channel;
let updatingFromChannel = false;

// Save text to local storage
const saveTextToLocal = (text) => {
  localStorage.setItem("text", text);
  console.log(`Saved to local storage: ${text}`);
};

// Subscribe to PieSocket channel
pieSocket
  .subscribe("text-room")
  .then((ch) => {
    channel = ch;
    console.log("Subscribed to channel:", channel);

    channel.listen("move", (data) => {
      console.log("Received data from channel:", data);
      const receivedData = JSON.parse(data);

      if (!updatingFromChannel) {
        updatingFromChannel = true;
        text.value = receivedData.text;
        saveTextToLocal(receivedData.text);
        updatingFromChannel = false;
      }
    });
  })
  .catch((error) => {
    console.error("Error subscribing to channel:", error);
  });

// Update text and publish to channel
const updateText = () => {
  console.log("Updating text:", text.value);
  saveTextToLocal(text.value);

  if (channel) {
    channel
      .publish("move", JSON.stringify({ text: text.value }))
      .then(() => {
        console.log("Successfully published to channel");
      })
      .catch((error) => {
        console.error("Error publishing to channel:", error);
      });
  }
};

onMounted(() => {
  console.log("Component mounted, initial text:", text.value);
});
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 1.5rem;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  margin-bottom: 1rem;
  font-size: 1.8rem;
  color: #333;
}

.label {
  display: block;
  margin-bottom: 0.5rem;
  font-size: 1rem;
  color: #555;
}

.text-area {
  width: 100%;
  height: 200px;
  padding: 0.75rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  resize: vertical;
  outline: none;
}

.text-area:focus {
  border-color: #007bff;
  box-shadow: 0 0 3px #007bff;
}

.update-button {
  display: block;
  width: 100%;
  padding: 0.75rem;
  margin-top: 1rem;
  font-size: 1rem;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  text-align: center;
}

.update-button:hover {
  background-color: #0056b3;
}

@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  h1 {
    font-size: 1.5rem;
  }

  .text-area {
    height: 150px;
  }
}
</style>
