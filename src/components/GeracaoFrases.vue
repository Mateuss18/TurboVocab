<template>
  <div>
    <input 
      v-model="inputWord" 
      placeholder="Digite uma palavra em inglês"
      @keyup.enter="fetchData"
    />
    <button @click="fetchData">Buscar</button>
    
    <div v-if="loading">Carregando...</div>
    
    <div v-if="translation">
      <h3>Tradução para Português:</h3>
      <p>{{ translation }}</p>
    </div>
    
    <div v-if="synonyms.length">
      <h3>Sinônimos em Português:</h3>
      <p>{{ synonyms.join(', ') }}</p>
    </div>
    
    <div v-if="sentences.length">
      <h3>Frases em Inglês:</h3>
      <ul>
        <li v-for="(sentence, index) in sentences" :key="index">{{ sentence }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

// Carrega a API Key das variáveis de ambiente
const API_KEY = import.meta.env.VITE_HUGGINFACE_API_KEY;
const inputWord = ref('');
const translation = ref(''); // Assumindo que já está funcionando
const synonyms = ref([]);
const sentences = ref([]);
const loading = ref(false);

// Função genérica para chamar a API via proxy
async function callInferenceAPI(model, prompt, params = {}) {
  const payload = {
    inputs: prompt,
    parameters: {
      max_length: params.max_length || 100,
      num_return_sequences: params.num_return_sequences || 1,
      do_sample: params.do_sample !== undefined ? params.do_sample : false,
      top_p: params.top_p || 0.9,
      top_k: params.top_k || 40,
    },
  };
  try {
    const response = await axios.post(
      `/api/models/${model}`, // Usando o proxy
      payload,
      { headers: { Authorization: `Bearer ${API_KEY}` } }
    );
    return response.data;
  } catch (error) {
    console.error(`Erro ao chamar o modelo ${model}:`, error);
    return [];
  }
}

async function fetchData() {
  const inputWord = { value: "bite" }; // Simulando sua entrada
  const sentences = []; // Array para armazenar as frases
  loading.value = true; // Ativa o indicador de carregamento

  // Faz 4 chamadas separadas à API
  for (let i = 0; i < 4; i++) {
    const sentencesPrompt = `Generate one simple English sentence containing the word "${inputWord.value}". Do not add any extra text or explanations.`;

    const sentencesData = await callInferenceAPI(
      'google/flan-t5-base',
      sentencesPrompt,
      {
        max_length: 70, // Limite reduzido para gerar uma frase curta
        num_return_sequences: 1, // Gera apenas uma sequência por chamada
        do_sample: true, // Ativa amostragem para variedade
        temperature: 0.7, // Controla a criatividade
        top_p: 0.9 // Controla a probabilidade cumulativa
      }
    );

    // Verifica se a frase foi gerada e contém a palavra
    if (sentencesData[0]?.generated_text) {
      const sentence = sentencesData[0].generated_text.trim();
      if (sentence.toLowerCase().includes(inputWord.value.toLowerCase()) && !sentences.includes(sentence)) {
        sentences.push(sentence); // Adiciona apenas se for única
      }
    }
  }

  // Garante que tenhamos exatamente 4 frases (caso contrário, exibe mensagem)
  if (sentences.length < 4) {
    console.warn("Não foi possível gerar 4 frases distintas.");
    sentences.value = sentences.length > 0 ? sentences : ["Frases não geradas"];
  } else {
    sentences.value = sentences; // Atualiza o ref com as frases geradas
  }

  loading.value = false; // Desativa o indicador de carregamento
  console.log("Frases em Inglês:");
  console.log(sentences.join('\n'));
}

fetchData();
</script>