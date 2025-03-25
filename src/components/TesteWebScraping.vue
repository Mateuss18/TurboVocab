<script setup>
import { ref } from "vue";
import axios from "axios";

const word = ref("");
const examples = ref([]);

const getExamples = async () => {
  if (!word.value) return;
  try {
    const { data } = await axios.get(
      `http://localhost:3000/scrape?word=${word.value}`
    );
    examples.value = data.examples;
  } catch (error) {
    console.error("Erro ao buscar exemplos:", error);
  }
};
</script>

<template>
  <div>
    <input
      v-model="word"
      placeholder="Digite uma palavra em inglês" />
    <button @click="getExamples">Buscar</button>

    <ul>
      <li
        v-for="(example, index) in examples"
        :key="index">
        {{ example }}
      </li>
    </ul>
  </div>
</template>

<style>
input,
button {
  margin: 5px;
  padding: 8px;
}
</style>
