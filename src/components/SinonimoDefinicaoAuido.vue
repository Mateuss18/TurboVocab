<template>
  <h1>Hello API</h1>

  <input
    v-model="inputWord"
    type="text" 
    @keyup.enter="getSinonimoDefinicaoAudio"
  />

  <br>

  <b>{{ inputWord }}</b>

  <p v-if="allSynonyms">
    Sinonimos: <br>
    {{ allSynonyms }}
  </p>

  <audio
    controls
    v-if="audioPronunciation"
    :src="audioPronunciation"></audio>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios'

let audioPronunciation = ref('')
let inputWord = ref('')
let allSynonyms = ref('')

async function getSinonimoDefinicaoAudio() {
  try {
    // RETORNA SINONIMOS, DEFINIÇÃO E AUDIO
    const response = await axios.get(`https://api.dictionaryapi.dev/api/v2/entries/en/${inputWord.value}`)

    const todosSinonimos = response.data[0].meanings.map(definicao => definicao.synonyms).flat()
    const primeirosSinonimos = response.data[0].meanings[0].synonyms

    allSynonyms.value = [...new Set([...todosSinonimos, ...primeirosSinonimos])]
    audioPronunciation.value = response.data[0].phonetics.filter(audios => audios.audio.includes('-us.'))[0].audio

    console.log('Audio: ', audioPronunciation.value);
    console.log('Retorno completo: ', response.data);
    console.log('SINONIMOS: ', allSynonyms.value);
    console.log(response.data[0].meanings.map(definicao => definicao.definitions));
    console.log(response.data[0].meanings[0].definitions.map(definicao => definicao.definition))

  } catch (error) {
    console.log(error)
  }
}
</script>
