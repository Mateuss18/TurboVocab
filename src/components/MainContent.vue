<script setup>
import { ref, onMounted } from "vue";

const vocabWord = ref(null);
const vocaWordList = ref([])

function addNewWord() {
  vocaWordList.value.push(vocabWord.value);
  vocabWord.value = null;

  console.log(vocaWordList.value);
  
}
</script>

<template>
  <header>
    <h1>TurboVocab</h1>
  </header>

  <form action="">
    <input
      v-model="vocabWord"
      type="text"
      name="vocabWord"
      id="vocabWord"
      placeholder="Insert the word here" 
    />

    <button @click.prevent="addNewWord" type="submit">Add</button>
  </form>

  <ul class="wordsList">
    <li v-for="word in vocaWordList">
      {{ word }}
    </li>

    <li>
      <b>Outside</b>
      <p>Fora, Lá fora, além</p>
      <ul>
        <li><b>Outside</b> food is not allowed in the theater during movies.</li>
        <li><b>Outside</b>, at the balcony, there was a young boy.</li>
        <li><b>PALAVRA</b> e frase</li>
      </ul>
    </li>

    <li>
      <b>Palavra</b>
      <p>Sinonimos da palavra</p>
      <ul>
        <li><b>PALAVRA</b> e frase</li>
      </ul>
    </li>
  </ul>
</template>

<style scoped></style>
