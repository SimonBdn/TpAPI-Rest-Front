<template>
  <main>
    <h1>Les clients</h1> <br>
    <div>
      <table>
        <caption>Liste des clients</caption>
        <tr>
          <th>Code</th>
          <th>Societe</th>
          <th>Contact</th>
          <th>Fonction</th>
        </tr>
        <!-- Si le tableau des catégories est vide -->
        <tr v-if="data.listeClients.length === 0">
          <td colspan="4">Veuillez patienter, chargement des clients...</td>
        </tr>
        <!-- Si le tableau des catégories n'est pas vide -->
        <tr v-for="clients in data.listeClients" :key="clients.code">
          <td>{{ clients.code }}</td>
          <td>{{ clients.societe }}</td>
          <td>{{ clients.contact }}</td>
          <td>{{ clients.fonction }}</td>
          <td>
            <button @click="deleteEntity(clients._links.self.href)">
              Supprimer
            </button>
          </td>
        </tr>
      </table>
    </div>
  </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest } from "@/api";

// Pour réinitialiser le formulaire
const categorieVide = {
  libelle: "",
  description: ""
};

let data = reactive({
  // Les données saisies dans le formulaire
  formulaireCategorie: { ...categorieVide },
  // La liste des catégories affichée sous forme de table
  listeClients: []
});

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}

function chargeClients() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest("/api/clients?page=0&size=5")
      .then((json) => {
        data.listeClients = json._embedded.clients;
      })
      .catch(showError);
}

function ajouteClients() {
  // Ajouter un client avec les données du formulaire
  const options = {
    method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
    body: JSON.stringify(data.formulaireClients),
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
  };
  doAjaxRequest("/api/clients", options)
      .then(() => {
        // Réinitialiser le formulaire
        data.formulaireCategorie = { ...categorieVide };
        // Recharger la liste des catégories
        chargeClients();
      })
      .catch(showError);
}
/**
 * Supprime une entité
 * @param entityRef l'URI de l'entité à supprimer
 */
function deleteEntity(entityRef) {
  doAjaxRequest(entityRef, { method: "DELETE", headers: { "Accept": "application/json" }})
      .then(chargeClients)
      .catch(showError);
}

// A l'affichage du composant, on affiche la liste
onMounted(chargeClients);

</script>


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
