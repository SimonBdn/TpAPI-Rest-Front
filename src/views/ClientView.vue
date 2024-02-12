<script setup>
import {onMounted, reactive} from "vue";
import {doAjaxRequest} from "@/api";

// Pour réinitialiser le formulaire
const clientsVide = {
  libelle: "",
  description: ""
};

const page = 0;

let data = reactive({
  // Les données saisies dans le formulaire
  formulaireCategorie: {...clientsVide},
  // La liste des catégories affichée sous forme de table
  listeCategories: [],
  listeLinks: []
});

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}
function chargeCategorieswithUrl(urlPage) {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  urlPage = urlPage.substring(urlPage.indexOf("/api"));
  doAjaxRequest(urlPage)
      .then((json) => {
        data.listeCategories = json._embedded.clients;
        data.listeLinks = json._links;
        console.log(data.listeLinks);
      })
      .catch(showError);
}
/*
function chargeCategories() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest("/api/clients?&page=" + page + "&size=5")
      .then((json) => {
        data.listeCategories = json._embedded.clients;
        data.listeLinks = json._links
        console.log(data.listeLinks)
      })
      .catch(showError);
}
*/


/**
 * Supprime une entité
 * @param entityRef l'URI de l'entité à supprimer
 */

// A l'affichage du composant, on affiche la liste
onMounted(() => {
  chargeCategorieswithUrl("/api/clients?&page=0&size=5");
})


</script>

<template>
  <div>
    <table>
      <caption>Liste des catégories</caption>
      <tr>
        <th>Code</th>
        <th>Société</th>
        <th>Contact</th>
        <th>Ville</th>
      </tr>
      <!-- Si le tableau des catégories est vide -->
      <tr v-if="data.listeCategories.length === 0">
        <td colspan="4">Veuillez patienter, chargement des catégories...</td>
      </tr>
      <!-- Si le tableau des catégories n'est pas vide -->
      <tr v-for="categorie in data.listeCategories" :key="categorie.code">
        <td>{{ categorie.code }}</td>
        <td>{{ categorie.societe }}</td>
        <td>{{ categorie.contact }}</td>
        <td>{{ categorie.adresse.ville }}</td>
      </tr>
      <tr>
        <td>
          <button type="number" @click="chargeCategorieswithUrl(data.listeLinks.first.href)">first</button>
        </td>
        <td>
          <button v-if="typeof data.listeLinks.prev !== 'undefined'" type="number" @click="chargeCategorieswithUrl(
              data.listeLinks.prev.href)">prev</button>
        </td>
        <td>
          <button v-if="typeof data.listeLinks.next !== 'undefined'" type="number" @click="chargeCategorieswithUrl(
              data.listeLinks.next.href)">next</button>
        </td>
        <td>
          <button type="number" @click="chargeCategorieswithUrl(data.listeLinks.last.href)">last</button>
        </td>
      </tr>
    </table>
  </div>

</template>

<style scoped>
td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #232623;
  color: rgb(255, 255, 255);
}
</style>