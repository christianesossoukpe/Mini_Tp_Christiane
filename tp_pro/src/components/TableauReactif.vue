<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

onMounted(() => {
  fetchProduits();
});
const isLoading = ref(false);
const itemsProduits = ref([]);
const error = ref(null);
const URL = "https://dummyjson.com/products";
const visibleColumn = ref([]);

const colonnes = computed(() => {
    if(itemsProduits.value.length === 0) return [];
    return Object.keys(itemsProduits.value[0]);
})
const fetchProduits = async () => {
  isLoading.value = true;
  error.value = null;
  try {
    const response = await axios.get(URL);
    const data = response.data;
    itemsProduits.value = data.products;
    console.log(itemsProduits.value);
  } catch (error) {
    error.value =
      "un probleme est survenu lors de la recuperation des données de produits";
  }
};
</script>

<template>
  <div>Tableau de produits dynamiques</div>
  <div v-if="isLoading">Chargement...</div>
  <div v-else>
    <div>
        <label for="colonnes"
        v-for="col in colonnes" :key="col">
    <input type="checkbox" value="col" name="checkbox"
    v-model="visibleColumn">
{{ col }}    
</label>
    </div>
    <table>
      <thead>
        <tr>
          <th v-for="(value, key) in itemsProduits[0]" :key="key">
            {{ key }}
          </th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>
</template>

<style scoped></style>
