<script setup lang="ts">
import { ref } from 'vue';
const currentWordIndex = ref(0)
const currentLetterIndex = ref(0)

type Status = 'corrent' | 'wrong' | 'extra' | null
type Letter = { letter: string, status: Status}
type Text = {
  letters: Letter[],
  error: boolean
}[]
const text = ref<Text>([
    {
      letters: [
        {letter: 'H', status: null},
        {letter: 'e', status: null},
        {letter: 'l', status: null},
        {letter: 'l', status: null},
        {letter: 'o', status: null},
      ],
      error: false
    },
    {
      letters: [
        {letter: 'V', status: null},
        {letter: 'i', status: null},
        {letter: 't', status: null},
        {letter: 'e', status: null},
      ],
      error: false
    },
    {
      letters: [
        {letter: 'T', status: null},
        {letter: 'y', status: null},
        {letter: 'p', status: null},
        {letter: 'e', status: null},
        {letter: 'S', status: null},
        {letter: 'c', status: null},
        {letter: 'r', status: null},
        {letter: 'i', status: null},
        {letter: 'p', status: null},
        {letter: 't', status: null},
      ],
      error: false
    }
]);

function pressLetter(event: KeyboardEvent){
  const key = event.key
  const letterIndex = currentLetterIndex.value
  const wordIndex = currentWordIndex.value
  const currentWord = text.value[wordIndex].letters
  const currentLetter = text.value[wordIndex].letters[letterIndex]
  if(isLetter(key)){
    if(letterIndex > currentWord.length - 1){
      text.value[wordIndex].letters.push({letter: key, status: 'extra'})
    }
    
    else if(key == currentLetter.letter){
      currentLetter.status = 'corrent'
    }
    else {
      currentLetter.status = 'wrong'
    }
    currentLetterIndex.value++ 
  }
  if(isBackSpace(event)){
    if(letterIndex){
      if(currentWord[letterIndex - 1].status == 'extra'){
        text.value[wordIndex].letters.pop()
      }
      else {
        currentWord[letterIndex - 1].status = null
      }
      currentLetterIndex.value--
    }
    else if(!letterIndex && wordIndex){
      currentWordIndex.value--
      const test = takeLastTypedLetter(text.value[wordIndex-1].letters)
      currentLetterIndex.value =  test + 1
      if(text.value[wordIndex-1].error) text.value[wordIndex-1].error = false
    }
  }
  if(isCtrlBackSpace(event)){
    if(!wordIndex) return
    if(!letterIndex){
      currentWordIndex.value--
    }
    const wordWithoutExtra = text.value[currentWordIndex.value].letters.filter(letter => letter.status != 'extra');
    const defaultText = wordWithoutExtra.map(({letter}) => ({letter, status: null}))
    text.value[currentWordIndex.value].letters = defaultText
    currentLetterIndex.value = 0
    if(text.value[currentWordIndex.value].error) text.value[currentWordIndex.value].error = false
  }
  if(isSpace(event.code)){
    if(wordIndex == text.value.length - 1) return
    if(!isAllLettersCorrect(text.value[wordIndex].letters)){
      text.value[wordIndex].error = true
    }
    currentWordIndex.value++
    currentLetterIndex.value = 0
  }
}
function isLetter(key: string){
  if(/^[a-z]$/i.test(key)) return true
  return false
}
function isBackSpace(event: KeyboardEvent){
  if(event.key === 'Backspace' && !event.ctrlKey) return true
  return false
}
function isCtrlBackSpace(event: KeyboardEvent){
  if(event.key == 'Backspace' && event.ctrlKey) return true
  return false
}

function isSpace(key: string){
  if(key === 'Space') return true
  return false
}
function isAllLettersCorrect(letters: Letter[]){
  return letters.every(letter => letter.status == 'corrent')
}
function takeLastTypedLetter(letters: Letter[]){
  for(let i = letters.length - 1; i >= 0; i--){
    if(letters[i].status){
      return i
    }
    if(!i){
      return 0
    }
  }
  return 0
}
document.addEventListener('keydown', pressLetter)
</script>
<template>
  <div style="display: flex; gap: 10px">
    <div v-for="({ letters, error }) in text">
      <span :class="{error, notEntered: !status, correctLetter: status == 'corrent', wrongLetter: status == 'wrong', extraLetter: status == 'extra'}" v-for="{letter, status} in letters">
        {{ letter }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.correctLetter {
  color: rgb(25, 124, 25);
}
.error {
  border-bottom: 1px solid red;
}
.notEntered {
  color: grey
}
.wrongLetter {
  color: red;
} 
.extraLetter {
  color: orange
}
</style>
