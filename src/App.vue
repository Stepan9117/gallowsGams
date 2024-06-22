<template>
  <GameHeader/>
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length"/>

    <GameWrongLetters :wrong-letters="wrongLetters"/>

    <GameWord :word="word" :correct-letters="correctLetters"/>
  </div>

  <GamePopup @restart="restart" :word="word" ref="popup"/>
  <GameNotification ref="notification"/>
</template>

<script setup lang="ts">
import GameHeader from "@/components/GameHeader.vue";
import GameFigure from "@/components/GameFigure.vue";
import GameWrongLetters from "@/components/GameWrongLetters.vue";
import GameWord from "@/components/GameWord.vue";
import GameNotification from "@/components/GameNotification.vue";
import GamePopup from "@/components/GamePopup.vue";
import {computed, ref, watch} from "vue";
import axios from "axios";

const getRandomWord = async () => {
  try {
    const {data} = await axios<{ FirstName: string }>
    ('https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )
    word.value = data.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}
getRandomWord()


const word = ref('')

const letters = ref<string[]>([]) //массив после стринг

const correctLetters = computed(() => letters.value.filter(x => word.value.includes(x))) // почему х
const wrongLetters = computed(() => letters.value.filter(x => !word.value.includes(x))) // почему х

const notification = ref<InstanceType<typeof GameNotification> | null>(null)

const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const restart = async () => {
  await getRandomWord()
  letters.value = []
  popup.value?.close()
}

const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every(x => correctLetters.value.includes(x)))

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {
  if (isWin.value) {//превратил в массив ...
    popup.value?.open('win')

  }

})

window.addEventListener("keydown", ({key}) => {
  if (isLose.value || isWin.value) {
    return
  }
  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})


</script>