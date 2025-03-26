<script setup>
import { ref, watch } from "vue";
import axios from "axios";

const word = ref("");
const examples = ref([]);
const translations = ref([]);
const displayTerm = ref("");
const isLoading = ref(false);
const error = ref(null);
const lastSearchedWord = ref("");

const getExamples = async () => {
  if (word.value.toLowerCase() === lastSearchedWord.value.toLowerCase()) {
    error.value = "Você já está buscando essa palavra";
    return;
  }

  if (!word.value.trim()) {
    error.value = "Por favor, digite uma palavra";
    return;
  }

  examples.value = [];
  translations.value = [];
  displayTerm.value = "";
  error.value = null;
  isLoading.value = true;

  try {
    const { data } = await axios.get(
      `http://localhost:3000/scrape?word=${encodeURIComponent(word.value)}`
    );

    examples.value = data.examples;
    translations.value = data.translations;
    displayTerm.value = data.displayTerm;

    lastSearchedWord.value = word.value.toLowerCase();
  } catch (err) {
    console.error("Erro ao buscar exemplos:", err);
    error.value = "Erro ao buscar exemplos. Tente novamente mais tarde.";
  } finally {
    isLoading.value = false;
  }
};

watch(word, (newVal) => {
  if (newVal.toLowerCase() !== lastSearchedWord.value.toLowerCase()) {
    error.value = null;
  }
});
</script>

<template>
  <div class="container">
    <h1>Exemplos de Uso de Palavras</h1>

    <div class="search-box">
      <input
        v-model.trim="word"
        placeholder="Digite uma palavra em inglês"
        @keyup.enter="getExamples" />
      <button
        @click="getExamples"
        :disabled="isLoading">
        {{ isLoading ? "Buscando..." : "Buscar" }}
      </button>
    </div>

    <div
      v-if="error"
      class="error-message">
      {{ error }}
    </div>

    <div
      v-if="displayTerm"
      class="term-display">
      <h2>Termo: "{{ displayTerm }}"</h2>
    </div>

    <div
      v-if="translations.length"
      class="translations">
      <h3>Traduções:</h3>
      <ul>
        <li
          v-for="(translation, index) in translations"
          :key="'trans-' + index">
          {{ translation }}
        </li>
      </ul>
    </div>

    <div
      v-if="examples.length"
      class="results">
      <h2>Exemplos para "{{ word }}"</h2>
      <ul>
        <li
          v-for="(example, index) in examples"
          :key="index">
          <p><strong>EN:</strong> {{ example.original }}</p>
          <p><strong>PT:</strong> {{ example.translation }}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.error-message {
  color: #ff4444;
  margin: 10px 0;
}

.results {
  margin-top: 20px;
}

.example-item {
  margin-bottom: 15px;
  padding: 10px;
  color: #000;
  background-color: #f5f5f5;
  border-left: 4px solid #42b983;
  white-space: pre-line;
}

h1,
h2 {
  color: #2c3e50;
}

.translations {
  margin: 20px 0;
}

.translations ul {
  list-style-type: none;
  padding: 0;
}

.translations li {
  padding: 8px;
  margin: 5px 0;
  color: #000;
  background-color: #e9f5ff;
  border-left: 4px solid #2196f3;
}

.term-display h2 {
  color: #2c3e50;
  margin-bottom: 15px;
}
</style>
