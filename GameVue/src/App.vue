<script setup>
import { ref } from "vue";

const messages = ref([]);
const userScore = ref(0);
const cpuScore = ref(0);

const apiUrl = "https://2ngrw7kq-3004.use2.devtunnels.ms/api/RPSgame/jugarRPS";

const determineWinner = (user, cpu) => {
  if (user === cpu) return "tie";

  if (
    (user === "piedra" && cpu === "tijera") ||
    (user === "papel" && cpu === "piedra") ||
    (user === "tijera" && cpu === "papel")
  ) {
    return "user";
  }

  return "cpu";
};

const play = async (choice) => {
  messages.value.push({ sender: "Tú", text: `Elijo ${choice}` });

  try {
    const url = `${apiUrl}?jugada=${encodeURIComponent(choice)}`;

    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text: choice }),
    });

    const data = await response.json();
    console.log(data);

    if (!data.cpu) {
      throw new Error("Respuesta inválida del servidor");
    }

    const cpuChoice = data.cpu;
    const winner = determineWinner(choice, cpuChoice);

    let resultText = `Yo elijo ${cpuChoice}.`;

    if (winner === "user") {
      userScore.value++;
      resultText += " ¡Ganaste!";
    } else if (winner === "cpu") {
      cpuScore.value++;
      resultText += " ¡Gané!";
    } else {
      resultText += " ¡Empate!";
    }

    setTimeout(() => {
      messages.value.push({
        sender: "CPU",
        text: resultText,
        winner: winner,
      });
    }, 1500);
  } catch (error) {
    console.error(error);
    setTimeout(() => {
      messages.value.push({
        sender: "CPU",
        text: "❌ Error conectando al servidor.",
      });
    }, 1500);
  }
};

const resetGame = () => {
  messages.value = [];
  userScore.value = 0;
  cpuScore.value = 0;
};
</script>

<template>
  <div id="app">
    <div class="container">
      <h1>Piedra, Papel, Tijeras</h1>

      <div class="score">
        <div class="score-item">Tú: {{ userScore }}</div>
        <div class="score-item">CPU: {{ cpuScore }}</div>
      </div>

      <div class="chat">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="message"
          :class="{
            user: msg.sender === 'Tú',
            cpu: msg.sender === 'CPU',
            win: msg.winner === 'user',
            lose: msg.winner === 'cpu',
            tie: msg.winner === 'tie',
          }"
        >
          <div class="avatar">{{ msg.sender === "Tú" ? "👤" : "🤖" }}</div>
          <div class="text">{{ msg.text }}</div>
        </div>
      </div>

      <div class="buttons">
        <button class="choice-btn" @click="play('piedra')">🪨 Piedra</button>
        <button class="choice-btn" @click="play('papel')">📄 Papel</button>
        <button class="choice-btn" @click="play('tijera')">✂️ Tijera</button>
      </div>

      <button class="reset-btn" @click="resetGame">Reiniciar Juego</button>
    </div>
  </div>
</template>

<style>
:root {
  font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #c9c2c2;

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

a {
  font-weight: 500;
  color: #646cff;
  text-decoration: inherit;
}
a:hover {
  color: #535bf2;
}

body {
  padding: auto;
  margin: 0;
  font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color-scheme: light dark;
  color: #213547;
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
  font-synthesis: none;
}

h1 {
  font-size: 3.2em;
  line-height: 1.1;
  text-align: center;
  margin-bottom: 20px;
  font-size: 2.5em;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #1a1a1a;
  cursor: pointer;
  transition: border-color 0.25s;
}
button:hover {
  border-color: #646cff;
}
button:focus,
button:focus-visible {
  outline: 4px auto -webkit-focus-ring-color;
}

.card {
  padding: 2em;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  width: 100vw;
  height: 100vh;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
}

.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 20px;
  box-sizing: border-box;
}

.chat {
  flex: 1;
  border: 2px solid rgba(129, 129, 129, 0.2);
  border-radius: 15px;
  overflow-y: auto;
  padding: 15px;
  margin-bottom: 20px;
  background: rgba(0, 0, 0, 0.701);
  backdrop-filter: blur(10px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.message {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
  animation: fadeIn 0.5s ease-in;
}

.message.user {
  justify-content: flex-end;
}

.message.cpu {
  justify-content: flex-start;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 10px;
  font-size: 1.5em;
}

.message.user .avatar {
  margin-right: 0;
  margin-left: 10px;
}

.text {
  background: rgba(255, 255, 255, 0.2);
  padding: 10px 15px;
  border-radius: 20px;
  max-width: 70%;
  word-wrap: break-word;
}

.message.user .text {
  background: rgba(0, 255, 0, 0.3);
}

.message.cpu .text {
  background: rgba(255, 0, 0, 0.3);
}

.message.win .text {
  background: rgba(0, 255, 0, 0.5);
  border: 2px solid #00ff00;
}

.message.lose .text {
  background: rgba(255, 0, 0, 0.5);
  border: 2px solid #ff0000;
}

.message.tie .text {
  background: rgba(255, 255, 0, 0.3);
  border: 2px solid #ffff00;
}

.buttons {
  display: flex;
  justify-content: space-around;
  gap: 10px;
}

.choice-btn {
  padding: 15px 25px;
  font-size: 18px;
  cursor: pointer;
  border: none;
  border-radius: 25px;
  background: rgba(156, 156, 156, 0.2);
  font-family: inherit;
  color: white;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.choice-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
  animation: pulse 1s infinite;
}

.choice-btn:active {
  transform: translateY(0);
}

.reset-btn {
  display: block;
  margin: 0 auto;
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border: none;
  border-radius: 25px;
  background: rgba(252, 2, 2, 0.71);
  color: white;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.reset-btn:hover {
  background: rgba(175, 6, 6, 0.699);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.reset-btn:active {
  transform: translateY(5px);
}

.score {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
  font-size: 1.2em;
  font-weight: bold;
}

.score-item {
  background: rgba(255, 255, 255, 0.1);
  padding: 10px 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
  a:hover {
    color: #747bff;
  }
  button {
    background-color: #f9f9f9;
  }
}
</style>
