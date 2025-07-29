<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'

import BotChat from './Chat/BotChat.vue'
import UserChat from './Chat/UserChat.vue'

interface Meal {
  id: number
  name: string
  category: string
  ingredients: string[]
  tags: string[]
  url: string
}

interface ChatMessage {
  type: 'bot' | 'user'
  content: string
  typing?: boolean
  actionType?: string
}

const isOpen = ref(false)
const chatMessages = ref<ChatMessage[]>([])

watch(isOpen, (value) => {
  if (value && chatMessages.value.length === 0) {
    chatMessages.value.push({
      type: 'bot',
      content:
      "Kamusta! Ako si Ma, ang taga-sagot sa tanong mong 'Anong ulam?' 💬 Halika, pili na tayo ng masarap!",
      typing: false,
    })
  }
})

const chatBody = ref<HTMLElement | null>(null)
watch(chatMessages, () => {
  nextTick(() => {
    if (chatBody.value) {
      chatBody.value.scrollTop = chatBody.value.scrollHeight
    }
  })
})

function removeTypingMessages() {
  chatMessages.value = chatMessages.value.filter((msg) => !msg.typing)
}

function replaceTypingWithMessage(message: ChatMessage) {
  const index = chatMessages.value.findIndex((msg) => msg.typing)
  if (index !== 1) {
    chatMessages.value.splice(index, 1, message)
  } else {
    chatMessages.value.push(message)
  }
}

function formatMeal(meal: Meal): string {
  return `Eto na ang random ulam mo today: ${meal.name}! 😋.
      Mga sangkap: ${meal.ingredients.join(', ')}.
      Kung gusto mo ng dagdag na info, check mo ito: ${meal.url} `
}

function formatMealList(meals: Meal[]): string {
  const mealNames = meals.map((meal) => meal.name).join(' , ')
  return `O siya! Heto ang mga pagkaing pampasaya buong linggo 🗓️:\n${mealNames}.`
}

function formatWeatherList(meals: Meal[], weather: string): string {
  const mealNames = meals.map((meal) => meal.name).join(' , ');

  const weatherEmojiMap: Record<string, string> = {
    rain: '🌧️',
    thunderstorm: '⛈️',
    clouds: '☁️',
    clear: '🌤️',
    drizzle: '🌦️',
  };

  const emoji = weatherEmojiMap[weather.toLowerCase()] || '☁️';

  return `The weather shows ${weather.toLowerCase()} in Manila ${emoji}, heto ang mga ulam na swak sa’yo:\n${mealNames}.`;
}

const handleButtonClick = (index: number) => {
  if (index === 0) {
    handleRecommendedClick()
  } else if (index === 1) {
    handleWeeklyPlanClick()
  } else if (index === 2) {
    handleWeatherClick()
  }
}

const fetchTodaysPick = async (): Promise<Meal> => {
  const response = await fetch('/api/menu/recommendation')
  if (!response.ok) {
    throw new Error('failed to fetch menus')
  }
  return await response.json()
}

const fetchWeeklyPlan = async (): Promise<Meal[]> => {
  const response = await fetch('/api/menu/weekly')
  if (!response.ok) {
    throw new Error('failed to fetch menus')
  }
  return await response.json()
}

type weatherMealResponse = {
  meals: Meal[]
  weather: string
}

const fetchWeatherMeals = async (): Promise<weatherMealResponse> => {
  const response = await fetch('/api/menu/weather')
  if (!response.ok) {
    throw new Error('failed to fetch menus')
  }
  const data = await response.json()
  return {
    meals: data.meals,
    weather: data.weather,
  }
}

const handleRecommendedClick = async () => {
  removeTypingMessages()

  chatMessages.value.push({
    type: 'user',
    content: 'Ma, random ulam reveal, go!',
  })

  chatMessages.value.push({
    type: 'bot',
    content: '',
    typing: true,
    actionType: 'todaysPick',
  })

  try {
    const data = await fetchTodaysPick()

    replaceTypingWithMessage({
      type: 'bot',
      content: formatMeal(data),
      actionType: 'todaysPick',
    })
  } catch (error) {
    replaceTypingWithMessage({
      type: 'bot',
      content: 'Oops! Can’t get your ulam picks right now. Try ulit later ha?'
    })
  }
}

const handleWeeklyPlanClick = async () => {
  removeTypingMessages()

  chatMessages.value.push({
    type: 'user',
    content: 'Anong masarap kainin buong linggo, Ma?',
  })

  chatMessages.value.push({
    type: 'bot',
    content: '',
    typing: true,
    actionType: 'weeklyPlan',
  })

  try {
    const data = await fetchWeeklyPlan()

    replaceTypingWithMessage({
      type: 'bot',
      content: formatMealList(data),
      actionType: 'weeklyPlan',
    })
  } catch (error) {
    replaceTypingWithMessage({
      type: 'bot',
      content: 'Oops! Can’t get your ulam picks right now. Try ulit later ha?'
    })
  }
}

const handleWeatherClick = async () => {
  removeTypingMessages()

  chatMessages.value.push({
    type: 'user',
    content: "Ma, anong ulam ang swak sa panahon ngayon?",
  })

  chatMessages.value.push({
    type: 'bot',
    content: '',
    typing: true,
    actionType: 'weatherMeals',
  })

  try {
    const { meals, weather } = await fetchWeatherMeals()

    replaceTypingWithMessage({
      type: 'bot',
      content: formatWeatherList(meals, weather),
      actionType: 'weatherMeals',
    })
  } catch (error) {
    replaceTypingWithMessage({
      type: 'bot',
      content: 'Oops! Can’t get your ulam picks right now. Try ulit later ha?'
    })
  }
}

const regenerateResponse = async (index: number) => {
  const message = chatMessages.value[index]
  const actionType = message?.actionType

  if (!actionType) return

  chatMessages.value[index] = {
    type: 'bot',
    content: '',
    typing: true,
    actionType,
  }

  try {
    if (actionType === 'weeklyPlan') {
      const data = await fetchWeeklyPlan()
      chatMessages.value[index] = {
        type: 'bot',
        content: formatMealList(data),
        actionType,
      }
    } else if (actionType === 'todaysPick') {
      const data = await fetchTodaysPick()
      chatMessages.value[index] = {
        type: 'bot',
        content: formatMeal(data),
        actionType,
      }
    } else if (actionType === 'weatherMeals') {
      const { meals, weather } = await fetchWeatherMeals()
      chatMessages.value[index] = {
        type: 'bot',
        content: formatWeatherList(meals, weather),
        actionType,
      }
    }
  } catch (error) {
    chatMessages.value[index] = {
      type: 'bot',
      content: 'Oops! Can’t get your ulam picks right now. Try ulit later ha?',
      actionType,
    }
  }
}
</script>

<template>
  <div>
    <div class="chatbot-container">
      <div class="icon-div" @click="isOpen = !isOpen">
        <img src="@/assets/girl.svg" alt="icon" class="chat-icon" />
      </div>

      <div v-if="isOpen" class="chat-window">
        <div class="chat-body" style="flex: 1; overflow-y: auto;" ref="chatBody">
          <template v-for="(msg, index) in chatMessages" :key="index">
            <BotChat
              v-if="msg.type === 'bot'"
              :key="index"
              :is-typing="msg.typing || false"
              :show-message="!msg.typing"
              :bot-message="msg.content"
              :has-button="index === 0"
              :has-smaller-button="index !== 0"
              :button-messages="[
                'Ma, random ulam reveal, go!',
                'Anong masarap kainin buong linggo, Ma?',
                'Ma, anong ulam ang swak sa panahon ngayon?',
              ]"
              @button-clicked="handleButtonClick"
              @regenerate-clicked="() => regenerateResponse(index)"
            />

            <UserChat v-else :bot-message="msg.content" />
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chatbot-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
}
.icon-div {
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  background-color: #e76f51;
  border-radius: 100%;
  width: 40px;
  height: 40px;
  z-index: 100;
  cursor: pointer;
}
.chat-icon {
  width: 30px;
  height: 30px;
}
.chat-window {
  position: fixed;
  bottom: 70px;
  right: 0;
  width: 400px;
  height: 500px;
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 1rem;
  display: flex;
  flex-direction: column;
}

</style>
