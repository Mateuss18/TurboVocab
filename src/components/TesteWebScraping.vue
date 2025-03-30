<template>
  <div class="container">
    <h1>TurboVocab</h1>

    <div class="search-box">
      <input
        v-model.trim="wordToLearn"
        placeholder="Digite uma palavra em inglês"
        @keyup.enter="getExamples" />
      <button
        @click="getExamples"
        :disabled="isLoading">
        {{ isLoading ? "Buscando..." : "Buscar" }}
      </button>
    </div>

    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>

    <div v-if="displayTerm" class="term-display">
      <h2>Termo: "{{ displayTerm }}"</h2>
    </div>

    <div v-if="translations.length" class="translations">
      <h3>Traduções:</h3>
      <div>
        <label
          v-for="(translation, index) in translations"
          :for="translation.index"
          :key="'trans-' + index"
          :class="{ 'selected-item': selectedTranslations.includes(translation) }"
        >
          <input 
            type="checkbox"
            :checked="selectedTranslations.includes(translation)"
            :id="translation.index"
            :name="translation" 
            :value="translation"
            v-model="selectedTranslations"
          >
          <span>{{ translation }}</span>
        </label>

        <button
         @click="copyTranslations" 
         class="btn-copy"
         :class="{ 'copyDisable': !selectedTranslations.length }" 
        >
         <img src="../assets/icons/icon-copy.svg" alt="">
        </button>
      </div>
    </div>

    <div v-if="examples.length" class="results">
      <h2>Exemplos para "{{ word }}"</h2>

      <ul>
        <li
          v-for="(example, index) in examples"
          :key="index"
        >
          <button @click="copyExemple(example.original)" title="Clique para copiar">
            {{ example.original }}
          </button>
          <p>{{ example.translation }}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";
import axios from "axios";
import { useToast } from "vue-toastification";

const toast = useToast()

const wordToLearn = ref("");
const word = ref("");
const examples = ref([]);
const translations = ref([]);
const displayTerm = ref("");
const isLoading = ref(false);
const errorMessage = ref(null);
const lastSearchedWord = ref("");

const selectedTranslations = ref([])
const copied = ref(false)

const formatTranslations = (translationsArray) => {
  return translationsArray
    .map(t => t.trim())
    .join(', ')
}

const getExamples = async () => {
  if (!wordToLearn.value.trim()) {
    errorMessage.value = "Por favor, digite uma palavra";
    return;
  }

  if (wordToLearn.value !== '' && wordToLearn.value.toLowerCase() === lastSearchedWord.value.toLowerCase()) {
    errorMessage.value = "Você já está buscando essa palavra";
    return;
  }

  setTimeout(() => {
    errorMessage.value = null;
  }, 2500);

  word.value = "";
  examples.value = [];
  translations.value = [];
  displayTerm.value = "";
  errorMessage.value = null;
  isLoading.value = true;

  try {
    const { data } = await axios.get(
      `http://localhost:3000/scrape?word=${encodeURIComponent(wordToLearn.value)}`
    );

    word.value = data.word
    examples.value = data.examples;
    translations.value = data.translations;
    displayTerm.value = data.displayTerm;

    lastSearchedWord.value = wordToLearn.value.toLowerCase();
  } catch (err) {
    console.error("Erro ao buscar exemplos:", err);
    errorMessage.value = "Erro ao buscar exemplos. Tente novamente mais tarde.";
  } finally {
    isLoading.value = false;
  }
};

setTimeout(() => {
  errorMessage.value = null;
}, 2500);

const copyTranslations = async () => {
  if (!selectedTranslations.value.length) {
    toast.error('Selecione pelo menos 1 opção')
    return
  }

  try {    
    await navigator.clipboard.writeText(formatTranslations(selectedTranslations.value))
    copied.value = true
    setTimeout(() => copied.value = false, 2000)

    toast.success(`Copiado \n ${formatTranslations(selectedTranslations.value)}`)
    selectedTranslations.value = []
  } catch (err) {
    console.error('Falha ao copiar: ', err)
  }
}

const copyExemple = (exempleToCopy) => {
  try {
    console.log(exempleToCopy);
    
    navigator.clipboard.writeText(exempleToCopy)
    toast.success(`Frase copiada \n ${exempleToCopy}`)
  } catch (err) {
    console.error('Falha ao copiar a frase: ', err)
  }
}

watch(wordToLearn, (newVal) => {
  if (newVal.toLowerCase() !== lastSearchedWord.value.toLowerCase()) {
    errorMessage.value = null;
  }
});
</script>

<style>
.container {
  margin: 0 auto;
  font-family: Arial, sans-serif;
}
.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.btn-copy {
  height: 40px;
  width: fit-content;
  padding: 9px;
}
.btn-copy img {
  width: 100%;
  height: 100%;
}
.btn-copy:hover {
  background-color: #2fac74;
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
.results ul {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 0;
}
.results li {
  display: flex;
  gap: 20px;
}
.results li button,
.results li p {
  text-align: left;
  width: 50%;
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
  color: #d4d4d4;
}
.translations {
  margin: 20px 0;
}
.translations div {
  display: flex;
  gap: 12px;
  list-style-type: none;
  padding: 0;
}
.translations .selected-item {
  background-color: #e9ffef;
  border-left: 4px solid #2cf321;
}
.translations label:hover {
  background-color: #e9ffef;
  border-left: 4px solid #2cf321;
}
.translations label {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px 8px;
  margin: 0;
  border-radius: 4px;
  color: #000;
  background-color: #ececec;
  border-left: 4px solid #787878;
  transition: 150ms all ease-in-out;
  min-width: 60px;
  line-height: 1.25;
  user-select: none
}
.translations label:hover {
  cursor: pointer;
}
.translations label input {
  display: none;
}
.term-display h2 {
  color: #ffffff;
  margin-bottom: 15px;
}

.copyDisable {
  background-color: gray;
  cursor: not-allowed;
}
.copyDisable:hover {
  background-color: #a3a3a3 !important;
}
</style>
