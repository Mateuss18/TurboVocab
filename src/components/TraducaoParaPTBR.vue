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
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

// Carrega a API Key das variáveis de ambiente
const API_KEY = import.meta.env.VITE_HUGGINFACE_API_KEY;
const inputWord = ref('');
const translation = ref('');
const synonyms = ref([]);
const sentences = ref([]);
const loading = ref(false);

/**
 * Função genérica para chamar a Inference API.
 * model: Nome do modelo (ex: 'google/flan-t5-base')
 * prompt: Texto de entrada
 * params: Parâmetros de geração
 */
async function callInferenceAPI(model, prompt, params = {}) {
  const payload = {
    inputs: prompt,
    parameters: {
      max_length: params.max_length || 50,
      num_return_sequences: params.num_return_sequences || 1,
      do_sample: params.do_sample !== undefined ? params.do_sample : false,
      top_p: params.top_p || 0.9,
      top_k: params.top_k || 40,
    }
  }
  try {
    const response = await axios.post(
      `/api/models/${model}`,
      payload,
      { headers: { Authorization: `Bearer ${API_KEY}` } }
    )
    return Array.isArray(response.data) ? response.data : [];
  } catch (error) {
    console.error(`Erro ao chamar o modelo ${model}:`, error);
    return [];
  }
}

async function fetchData() {
  if (!inputWord.value) return;
  loading.value = true;
  translation.value = '';
  synonyms.value = [];
  sentences.value = [];
  
  try {
    // 1. Tradução (usando o modelo Helsinki-NLP/opus-mt-tc-big-en-pt)
    const translationData = await callInferenceAPI(
      'Helsinki-NLP/opus-mt-tc-big-en-pt',
      inputWord.value,
      { max_length: 30, num_return_sequences: 1, do_sample: false }
    );
    if (translationData.length && translationData[0].translation_text) {
      translation.value = translationData[0].translation_text.trim();
    } else {
      translation.value = 'Tradução não disponível';
    }

    
  } catch (error) {
    console.error('Erro durante o processamento:', error);
  } finally {
    loading.value = false;
  }
}
</script>
