<template>
  <div>
    <h2>Exemplo de Tradução com Apertium</h2>
    <input v-model="texto" placeholder="Digite o texto para traduzir" />

    <button @click="traduzir">Traduzir</button>

    <div v-if="textoTraduzido">
      <h3>Texto Traduzido:</h3>
      <p>{{ textoTraduzido }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const texto = ref("");
const source = ref("eng");
const target = ref("por");
const textoTraduzido = ref("");

const traduzir = async () => {
  if (!texto.value) {
    alert("Digite um texto para traduzir.");
    return;
  }

  try {
    const response = await fetch(
      `https://beta.apertium.org/apy/translate?q=${encodeURIComponent(texto.value)}&langpair=${source.value}|${target.value}`
    );    

    if (!response.ok) {
      throw new Error(`Erro: ${response.status} - ${response.statusText}`);
    }

    const data = await response.json();

    textoTraduzido.value = data.responseData.translatedText;
  } catch (error) {
    console.error("Erro ao traduzir:", error);
    alert("Erro ao traduzir. Tente novamente mais tarde.");
  }
};
</script>

<style scoped>
input,
select {
  margin: 0.5rem 0;
  padding: 0.5rem;
  font-size: 1rem;
}
button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
}
</style>
