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
const words = ref<HTMLDivElement[]>([])
const x = ref(0)
const y = ref(0)
const getY = () => {
  const y = words.value[currentWordIndex.value]?.offsetTop
  return y
}
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
    x.value += getLetterWidth(words.value[wordIndex]?.children[0])
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
      x.value -= words.value[wordIndex]?.children[0].clientWidth!
      currentLetterIndex.value--
    }
    else if(!letterIndex && wordIndex){
      currentWordIndex.value--
      const test = takeLastTypedLetter(text.value[wordIndex-1].letters)
      currentLetterIndex.value = test + 1
      if(text.value[wordIndex-1].error) text.value[wordIndex-1].error = false
      //to separate function
      const element = words.value[wordIndex-1].children
      const children  = Array.from(element)
      for(let letterIndex = children.length; letterIndex > 0; letterIndex--){
        const letter = children[letterIndex] as HTMLElement
        if(letter && !letter?.className.includes('notEntered')){
          const wordOffset = words.value[wordIndex-1].offsetLeft
          x.value = wordOffset + letter.offsetLeft + letter.offsetWidth
          break
        }
      }
    }
  }
  if(isCtrlBackSpace(event)){
    if(!wordIndex && !letterIndex) return
    if(!letterIndex){
      currentWordIndex.value--
    }
    const wordWithoutExtra = text.value[currentWordIndex.value].letters.filter(letter => letter.status != 'extra');
    const defaultText = wordWithoutExtra.map(({letter}) => ({letter, status: null}))
    text.value[currentWordIndex.value].letters = defaultText
    currentLetterIndex.value = 0
    x.value = words.value[currentWordIndex.value]?.offsetLeft
    if(text.value[currentWordIndex.value].error) text.value[currentWordIndex.value].error = false
  }
  if(isSpace(event.code)){
    if(wordIndex == text.value.length - 1) return
    if(!isAllLettersCorrect(text.value[wordIndex].letters)){
      text.value[wordIndex].error = true
    }
    currentWordIndex.value++
    currentLetterIndex.value = 0
    x.value = words.value[currentWordIndex.value]?.offsetLeft
  }
}
function getLetterWidth(letter: HTMLElement | Element){
  return letter.getBoundingClientRect().width
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
  <div style="font-size: 24px;">
    <div style="display: flex; gap: 1em; position: relative">
    <div 
      :style="{
        top: getY() + 'px',
        left: x + 'px'
      }"
      class="caret"></div>
      <div 
        ref="words"
        class="word"
        :class="{
          error,
          active: wordIndex == currentWordIndex
        }"
        v-for="({ letters, error }, wordIndex) in text">
        <div 
          v-for="{letter, status} in letters" 
          class="letter"
          :class="{
            notEntered: !status, 
            correctLetter: status == 'corrent', 
            wrongLetter: status == 'wrong', 
            extraLetter: status == 'extra',
            }">
          {{ letter }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.correctLetter {
  color: rgb(25, 124, 25);
}
.caret {
  position: absolute;
  width: 3px;
  height: 29px;
  background-color: rgb(24, 130, 156);
  border-radius: 5px;
  animation: pulse 1.2s infinite;
  transition-duration: 0.1s;
  z-index: 10;
}
.word {
  display: flex;
  user-select: none;
  position: relative
}
.letter {
  height: 1.2em;
  display: flex;
  align-items: center;
}
.error {
  border-bottom: 1px solid red;
}
.leftIndicator {
  border-left: 1px solid pink;
}
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}
.rightIndicator:last-child {
  border-right: 1px solid pink;
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
