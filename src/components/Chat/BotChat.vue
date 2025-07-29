<script setup lang="ts">
  import { defineEmits } from 'vue';
  import ArrowPathIcon from '../icons/ArrowPathIcon.vue';

  const props = defineProps<{
    isTyping?: boolean,
    showMessage?: boolean,
    botMessage?: string,
    hasButton?:boolean,
    hasSmallerButton?:boolean,
    buttonMessages?: string[];
  }>();

  const emit = defineEmits(['button-clicked', 'regenerate-clicked']);

  function handleClick(index: number) {
    emit('button-clicked', index);
  }

  function handleRegenerateClick() {
    emit('regenerate-clicked');
  }

</script>

<template>
  <div class="bot-chat">
    <img src="@/assets/girl.svg" alt="icon" class="botchat-icon" />
      <div class="chat-content">
        <div v-if="props.isTyping" class="typing-indicator">
          <p><span class="dot one">.</span><span class="dot two">.</span><span class="dot three">.</span></p>
        </div>
        <div v-else-if="props.showMessage">
          <p style="font-size: 12px;">Ma</p>
            <div class="content">
              <p>{{ props.botMessage }}</p>
            </div>
            <div v-if="hasButton">
              <button
                v-for="(message,index) in buttonMessages"
                :key="index"
                @click="handleClick(index)"
                class="choice"
              >
                {{ message }}
              </button>
            </div>
            <div v-if="hasSmallerButton" @click="handleRegenerateClick()" class="regenerate-div">
              <button key="index"
                class="small-choice"
              >
              <ArrowPathIcon class="arrowpath-icon" />
                Regenerate response
              </button>
            </div>
          </div>
        </div>
  </div>
</template>

<style scoped>

  .botchat-icon {
    width: 30px;
    height: 30px;
    flex-shrink: 0;
    margin-right: 5px;
  }
  .content {
    background-color: #F5F5F5;
    padding: 5px;
    border-radius: 5px;
  }
  .bot-chat {
    display: flex;
    align-items: flex-end;
    margin-top: 20px;
  }
  .typing-indicator .dot {
    display: inline-block;
    animation: bounce 1.4s infinite ease-in-out;
    font-weight: bold;
    font-size: 1.2em;
    padding: 2px;
    font-weight: 700;
  }
  .typing-indicator .dot.one {
    animation-delay: 0s;
  }
  .typing-indicator .dot.two {
    animation-delay: 0.2s;
  }
  .typing-indicator .dot.three {
    animation-delay: 0.4s;
  }
  .choice {
    font-size: 13px;
    padding: 5px;
    margin-top: 5px;
    width: 100%;
    background: none;
    border: 1px solid rgba(231, 111, 81, 0.8);
    border-radius: 5px;
    cursor: pointer;
    color: black;
  }
  .regenerate-div {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 5px;
  }
  .arrowpath-icon {
    width: 12px;
    height: 12px;
  }
  .small-choice {
    display: flex;
    align-items: center;
    font-size: 12px;
    padding: 4px;
    margin-top: 5px;
    background: none;
    border: none;
    cursor: pointer;
    gap: 5px;
    color: black;
  }
  .small-choice:hover {
    border-bottom: 1px solid #E76F51;
  }
  .choice:hover {
    background-color: rgba(231, 111, 81, 0.8);
    color: white;
  }

@keyframes bounce {
  0%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-8px);
  }
}


</style>
