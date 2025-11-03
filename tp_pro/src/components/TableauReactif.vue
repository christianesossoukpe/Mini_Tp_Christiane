<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

// État de chargement
const isLoading = ref(false);

// etat pour recuperer données des produits
const itemsProduits = ref([]);
// etat de gestion des erreurs de recuperation des donnees
const error = ref(null);
// etat pour les colonnes visible
const visibleColumn = ref([]);

// URL API
const URL = "https://dummyjson.com/products";

// utilisation d'axios pour recuperer les donnees de l'api
const fetchProduits = async () => {
  isLoading.value = true;
  error.value = null;

  try {
    const response = await axios.get(URL);
    const data = response.data;
    console.log(data);
    // Vérifie que data.products est un tableau
    itemsProduits.value = Array.isArray(data.products) ? data.products : [];

    if (itemsProduits.value.length > 0) {
      visibleColumn.value = Object.keys(itemsProduits.value[0]);
    }
  } catch (err) {
    error.value =
      "Un problème est survenu lors de la récupération des données.";
  } finally {
    isLoading.value = false;
  }
};

// Colonnes dynamiques du tableau à afficher pour permettre de choisir les cases à cocher ou non
const colonnes = computed(() => {
  if (itemsProduits.value.length === 0) return [];
  return Object.keys(itemsProduits.value[0]);
});

// recuperation des donnees au montage  du composant
onMounted(() => {
  fetchProduits();
});
</script>

<template>
  <div style="padding: 20px">
    <!-- <h2>🛒 Tableau de produits dynamiques</h2> -->

    <div v-if="isLoading" class="status">Chargement...</div>
    <div v-else-if="error" class="status">{{ error }}</div>

    <div v-else>
      <!-- Filtres de colonnes de choix -->
      <div v-if="colonnes.length" style="margin-bottom: 1rem">
        <strong>Afficher les colonnes de la table des produits :</strong>
        <div
          v-for="col in colonnes"
          :key="col"
          style="display: inline-block; margin-right: 10px"
        >
          <label>
            <input type="checkbox" :value="col" v-model="visibleColumn" />
            {{ col }}
          </label>
        </div>
      </div>

      <!-- Tableau  -->
      <table>
        <thead>
          <tr>
            <th v-for="col in visibleColumn" :key="col">{{ col }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="!itemsProduits.length">
            <td :colspan="visibleColumn.length">Aucun produit disponible</td>
          </tr>

          <tr v-for="(item, index) in itemsProduits" :key="index">
            <td v-for="col in visibleColumn" :key="col">
              <template v-if="col === 'dimensions' && item[col]">
                📏 {{ item[col].width }}x{{ item[col].height }}x{{
                  item[col].depth
                }}
              </template>

              <template
                v-else-if="col === 'reviews' && Array.isArray(item[col])"
              >
                {{ item[col].length }} avis
              </template>

              <template v-else-if="col === 'meta' && item[col]">
                🏷️ {{ item[col].barcode || "N/A" }}
              </template>
              <template v-else-if="col === 'tags' && Array.isArray(item[col])">
                🏷️ {{ item[col].join(", ") }}
              </template>
               <template v-else-if="col === 'images' && Array.isArray(item[col])">
                <div class="images-list">
                  <img
                    v-for="(img, i) in item[col].slice(0, 3)"  
                    :key="i"
                    :src="img"
                    alt="Image du produit"
                    class="image-mini"
                  />
                </div>
              </template>
              <template v-else-if="typeof item[col] === 'object'">
                {{ JSON.stringify(item[col]) }}
              </template>

              <template v-else>
                {{ item[col] }}
              </template>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<style scoped>
/* Styles du tableau */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

/* Style des cellules et en-têtes */
th,
td {
  border: 1px solid #000; /* bordure noire */
  padding: 8px;
  text-align: left;
}

/* En-tête du tableau */
thead th {
  background-color: #f2f2f2;
}

/* Labels et cases à cocher */
label {
  font-family: Arial, sans-serif;
  font-size: 14px;
}

input[type="checkbox"] {
  margin-right: 5px;
  cursor: pointer;
}

/* Message de chargement ou d'erreur */
.status {
  font-weight: bold;
  margin-bottom: 10px;
}
.image-mini {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 6px;
  margin-right: 5px;
  border: 1px solid #ccc;
}
</style>
