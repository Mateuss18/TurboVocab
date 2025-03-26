<script setup>
import { ref } from "vue";
import axios from 'axios'

const vocabWord = ref(null);
const vocaWordList = ref([]);
const wordsApiUrl = import.meta.env.VITE_WORDS_API_KEY;
const memoryApiUrl = import.meta.env.VITE_MY_MEMORY_API_KEY;
const resultSinonimos = ref([])
let sinonimosTraduzidos = ref([])

async function addNewWord() {
  if (vocabWord.value !== null) {
    await buscarSinonimos(vocabWord.value)

    console.log('SINONIMOS: ' + resultSinonimos.value);

    for (let index = 0; index < resultSinonimos.value.length; index++) {
      sinonimosTraduzidos.value.push(await translateToPortuguese(resultSinonimos.value[index]))
    }

    console.log('TRADUZIDOS: ' + sinonimosTraduzidos.value);
  }
}

const buscarSinonimos = async (word) => {
  try {
    const responseSinonimos = await axios.get(
      `https://wordsapiv1.p.rapidapi.com/words/${word}/synonyms`,
      {
        params: {
          limit: 5
        },
        headers: {
          'X-RapidAPI-Key': `${wordsApiUrl}`,
          'X-RapidAPI-Host': 'wordsapiv1.p.rapidapi.com'
        }
      }
    )

    // console.log(responseSinonimos.data.synonyms);
    
    resultSinonimos.value = responseSinonimos.data.synonyms.slice(0, 5)
  } catch (error) {
    console.error(error)
  }
}

const translateToPortuguese = async (word) => {
  try {
    const responseTranslation = await axios.get(
      `https://api.mymemory.translated.net/get?q=${word}&langpair=en|pt`
    )

    if(!responseTranslation.data.quotaFinished && responseTranslation.data.responseData.match > 0.84) {
      return responseTranslation.data.responseData.translatedText
    }

  } catch (error) {
    console.error(error)
  } 
}

</script>

<template>
  <header>
    <h1>TurboVocab</h1>
  </header>

  <form>
    <input
      required
      v-model="vocabWord"
      type="text"
      name="vocabWord"
      id="vocabWord"
      placeholder="Insert the word here" 
    />

    <button @click.prevent="addNewWord" type="submit">Add</button>
  </form>

  <div class="wordsList">
    <div v-if="sinonimosTraduzidos.length">
      <h2>SINONIMOS</h2>
      <div>
        <span v-for="sinonimo in sinonimosTraduzidos">
          {{ sinonimo }} &#8202;
        </span>
      </div>
    </div>

    <li style="margin-top: 50px;">
      <b>Word: Outside</b>
      <p>Sinonimos: Fora, Lá fora, além</p>
      <ul>
        <li>Frases:</li>
        <li><b>Outside</b> food is not allowed in the theater during movies.</li>
        <li><b>Outside</b>, at the balcony, there was a young boy.</li>
        <li><b>PALAVRA</b> e frase</li>
      </ul>
    </li>
  </div>
</template>