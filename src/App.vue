<script setup lang="ts">
import { Ref, ref, toValue } from 'vue';
const wordIndex = ref(0)
const letterIndex = ref(0)

type Status = 'corrent' | 'wrong' | 'extra' | null
type Letter = { letter: string, status: Status}
type Word = {
  letters: Letter[],
  error: boolean
}
const words = ref<HTMLDivElement[]>([])
const x = ref(0)
const y = ref(0)
const text = ref<Word[]>([
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
  const letters = getWordFromText(text, wordIndex).letters
  const currentLetter = getLetter(letters, letterIndex)
  if(isLetter(key)){
    if(toValue(letterIndex) > letters.length - 1){
      const letters = getWordFromText(text, wordIndex).letters
      letters.push({letter: key, status: 'extra'})
    }
    else if(key == currentLetter.letter){
      currentLetter.status = 'corrent'
    }
    else {
      currentLetter.status = 'wrong'
    }
    x.value += getElementWidth(toValue(words)[toValue(wordIndex)]?.children[0])
    letterIndex.value++ 
  }
  if(isBackSpace(event)){
    if(toValue(letterIndex)){
      const prevLetter = getPrevLetter(letters, letterIndex)
      if(prevLetter.status == 'extra'){
        const word = getWordFromText(text, wordIndex)
        word.letters.pop()
      }
      else {
        prevLetter.status = null
      }
      const letterElement = getWordElementFromText(words, wordIndex).children[0]!
      const wordElementWidth = getElementWidth(letterElement)
      x.value -= wordElementWidth
      letterIndex.value--
    }
    else if(toValue(wordIndex)){
      wordIndex.value--
      const word = getWordFromText(text, wordIndex)
      resetCurrentWordError(word)
      const letters = word.letters
      const lastTypedLetterIndex = takeLastTypedLetterIndex(letters)
      letterIndex.value = lastTypedLetterIndex + 1
      const wordElement = getWordElementFromText(words, wordIndex)
      const letterElements = Array.from(wordElement.children)
      const lastTypedLetter = letterElements[lastTypedLetterIndex]
      x.value = getOffsetLeft(wordElement) + getOffsetLeft(lastTypedLetter) + getElementWidth(lastTypedLetter)
    }
  }
  if(isCtrlBackSpace(event)){
    if(!wordIndex.value && !letterIndex.value) return
    if(!letterIndex.value){
      wordIndex.value--
    }
    const word = getWordFromText(text, wordIndex)
    const wordWithoutExtra = word.letters.filter(letter => letter.status != 'extra');
    const defaultText = wordWithoutExtra.map(({letter}) => ({letter, status: null}))
    word.letters = defaultText
    letterIndex.value = 0
    const wordElement = getWordElementFromText(words, wordIndex)
    x.value = wordElement.offsetLeft
    resetCurrentWordError(word)
  }
  if(isSpace(event.code)){
    if(toValue(wordIndex) == text.value.length - 1) return
    if(!isAllLettersCorrect(getWordFromText(text, wordIndex).letters)){
      getWordFromText(text, wordIndex).error = true
    }
    wordIndex.value++
    letterIndex.value = 0
    const wordElement = getWordElementFromText(words, wordIndex)
    x.value = wordElement.offsetLeft
    y.value = wordElement.offsetTop
  }
}
function getWordFromText(text: Ref<Word[]>, wordIndex: Ref<number>){
  return text.value[wordIndex.value]
}
function getWordElementFromText(text: Ref<HTMLDivElement[]>, wordIndex: Ref<number>){
  return text.value[wordIndex.value]
}
function resetCurrentWordError(word: Word){
  if(word.error) word.error = false
}
function getLetter(letters: Letter[], index: Ref<number>){
  return letters[index.value]
}
function getPrevLetter(letters: Letter[], index: Ref<number>){
  return letters[index.value - 1]
}
function getElementWidth(element: HTMLElement | Element){
  return element.getBoundingClientRect().width
}
function getOffsetLeft(element: HTMLElement | Element){
  if(element instanceof HTMLElement) return element.offsetLeft
  return (element as HTMLElement).offsetLeft
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
function takeLastTypedLetterIndex(letters: Letter[]){
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
    <div style="display: flex; gap: 10px; position: relative">
    <div 
      :style="{
        top: y + 'px',
        left: x + 'px'
      }"
      class="caret"></div>
      <div 
        ref="words"
        class="word"
        :class="{
          error,
          active: wordIndex == wordIndex
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
