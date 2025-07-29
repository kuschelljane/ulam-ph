<script setup lang="ts">
  const props = defineProps<{
    botMessage?: string,
  }>();
</script>

<template>
  <div class="bot-chat">
      <div class="chat-content">
        <div>
          <p class="chat-name">You</p>
            <div class="content">
              <p>{{ props.botMessage }}</p>
            </div>
          </div>
        </div>
  </div>
</template>

<style scoped>
  .chat-name {
    font-size: 12px;
    display:flex;
    justify-content: flex-end;
  }
  .content {
    background-color: rgba(231, 111, 81, 0.8);
    padding: 5px;
    border-radius: 5px;
    color: white;
  }
  .bot-chat {
    display: flex;
    justify-content: flex-end;
    margin-top: 20px;
  }
</style>
