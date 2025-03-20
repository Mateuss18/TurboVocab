<template>
  <h1>Hello API</h1>
</template>

<script setup>
import axios from 'axios'

const word = 'foundation'

async function testeAPI() {
  try {
    // RETORNA SINONIMOS, DEFINIÇÃO E AUDIO
    const response = await axios.get(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`)

    console.log('Retorno completo: ', response.data[0].meanings);
    console.log(response.data[0].meanings.map(definicao => definicao.definitions));
    console.log('Todos os sinonimos: ', response.data[0].meanings.map(definicao => definicao.synonyms).flat());
    console.log('Sinonimo: ', response.data[0].meanings[0].synonyms);
    console.log(response.data[0].meanings[0].definitions.map(definicao => definicao.definition))
    
  } catch (error) {
    console.log(error)
  }
}

testeAPI()
</script>