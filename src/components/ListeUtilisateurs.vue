<template>
  <div class="page">
    <div v-if="page === 'ajouter'">
      <button class="btn-retour" @click="page = 'liste'; chargerUtilisateurs()">
        ← Retour à la liste
      </button>
      <AjouterUtilisateur @utilisateur-ajoute="page = 'liste'; chargerUtilisateurs()" />
    </div>

    <div v-else-if="page === 'modifier'" class="vue-modifier">
      <button class="btn-retour" @click="page = 'liste'">
        ← Retour à la liste
      </button>

      <h2>Modifier l'utilisateur</h2>

      <div>
        <label>ID : </label>
        <input type="text" :value="idModif" disabled />
      </div>

      <div>
        <label>Nom : </label>
        <input type="text" v-model="nomModif" />
      </div>

      <div>
        <label>Prénom : </label>
        <input type="text" v-model="prenomModif" />
      </div>

      <div>
        <button class="btn-valider" @click="enregistrer">Enregistrer</button>
        <button class="btn-annuler" @click="page = 'liste'">Annuler</button>
      </div>
    </div>

    <div v-else-if="page === 'supprimer'" class="confirmation">
      <h2>Confirmer la suppression</h2>
      <p>
        Voulez-vous vraiment supprimer l'utilisateur
        <strong>{{ utilisateurASupprimer?.prenom }} {{ utilisateurASupprimer?.nom }}</strong>
        (ID : {{ utilisateurASupprimer?.id }}) ?
      </p>
      <div>
        <button class="btn-supprimer" @click="confirmerSuppression">Confirmer</button>
        <button class="btn-annuler" @click="annulerSuppression">Annuler</button>
      </div>
    </div>

    <div v-else>
      <div class="header">
        <h2>Liste des utilisateurs</h2>
        <button class="btn-plus" @click="page = 'ajouter'">+ Ajouter</button>
      </div>

      <div class="zone-filtres">
        <div class="barre-recherche">
          <label>Rechercher : </label>
          <input
            type="text"
            v-model="recherche"
            placeholder="Rechercher par nom ou prénom..."
          />
        </div>

        <div class="select-filtre">
          <label>Filtrer par état : </label>
          <select v-model="filtreEtat">
            <option value="tous">Tous les utilisateurs</option>
            <option value="actifs">Uniquement les actifs</option>
            <option value="inactifs">Uniquement les inactifs</option>
          </select>
        </div>
      </div>

      <table border="1">
        <thead>
          <tr>
            <th>ID</th>
            <th>Nom</th>
            <th>Prénom</th>
            <th>État</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="u in utilisateursFiltres" :key="u.id">
            <td>{{ u.id }}</td>
            <td>{{ u.nom }}</td>
            <td>{{ u.prenom }}</td>
            <td>
              <span :class="['badge', u.actif ? 'badge-actif' : 'badge-inactif']">
                {{ u.actif ? 'Actif' : 'Inactif' }}
              </span>
              <button class="btn-toggle" @click="basculerEtat(u)">
                {{ u.actif ? 'Désactiver' : 'Activer' }}
              </button>
            </td>
            <td>
              <button class="btn-modifier" @click="demarrerModif(u)">Modifier</button>
              <button class="btn-supprimer" @click="demanderSuppression(u)">Supprimer</button>
            </td>
          </tr>
          <tr v-if="utilisateursFiltres.length === 0">
            <td colspan="5" class="aucun-resultat">Aucun utilisateur trouvé</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import AjouterUtilisateur from './AjouterUtilisateur.vue'

const page = ref<'liste' | 'ajouter' | 'modifier' | 'supprimer'>('liste')
const utilisateurs = ref<any[]>([])
const recherche = ref('')
const filtreEtat = ref<'tous' | 'actifs' | 'inactifs'>('tous')

const utilisateurASupprimer = ref<any>(null)

const idModif = ref<number | string | null>(null)
const nomModif = ref('')
const prenomModif = ref('')
const actifModif = ref(true)

const utilisateursFiltres = computed(() => {
  return utilisateurs.value.filter(u => {
    if (filtreEtat.value === 'actifs' && !u.actif) {
      return false
    }
    if (filtreEtat.value === 'inactifs' && u.actif) {
      return false
    }

    if (recherche.value.trim()) {
      const motCle = recherche.value.toLowerCase()
      const correspondNom = u.nom && u.nom.toLowerCase().includes(motCle)
      const correspondPrenom = u.prenom && u.prenom.toLowerCase().includes(motCle)
      if (!correspondNom && !correspondPrenom) {
        return false
      }
    }

    return true
  })
})

function chargerUtilisateurs() {
  fetch('http://localhost:3000/users')
    .then(res => res.json())
    .then(data => {
      utilisateurs.value = data
    })
}

function basculerEtat(u: any) {
  fetch('http://localhost:3000/users/' + u.id, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      actif: !u.actif
    })
  }).then(() => {
    chargerUtilisateurs()
  })
}

function demarrerModif(u: any) {
  idModif.value = u.id
  nomModif.value = u.nom
  prenomModif.value = u.prenom
  actifModif.value = u.actif ?? true
  page.value = 'modifier'
}

function enregistrer() {
  if (idModif.value !== null) {
    fetch('http://localhost:3000/users/' + idModif.value, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        id: idModif.value,
        nom: nomModif.value,
        prenom: prenomModif.value,
        actif: actifModif.value
      })
    }).then(() => {
      idModif.value = null
      page.value = 'liste'
      chargerUtilisateurs()
    })
  }
}

function demanderSuppression(u: any) {
  utilisateurASupprimer.value = u
  page.value = 'supprimer'
}

function confirmerSuppression() {
  if (utilisateurASupprimer.value) {
    fetch('http://localhost:3000/users/' + utilisateurASupprimer.value.id, {
      method: 'DELETE'
    }).then(() => {
      utilisateurASupprimer.value = null
      page.value = 'liste'
      chargerUtilisateurs()
    })
  }
}

function annulerSuppression() {
  utilisateurASupprimer.value = null
  page.value = 'liste'
}

onMounted(() => {
  chargerUtilisateurs()
})
</script>

<style scoped>
.page {
  background-color: #ffffff;
  color: #333333;
  padding: 20px;
  min-height: 100vh;
}

.zone-filtres {
  display: flex;
  gap: 20px;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

.barre-recherche label,
.select-filtre label {
  font-weight: bold;
  margin-right: 8px;
}

.barre-recherche input,
.select-filtre select {
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.barre-recherche input {
  width: 250px;
}

.aucun-resultat {
  text-align: center;
  color: #888;
  padding: 15px;
}

.badge {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
  color: white;
  margin-right: 8px;
}

.badge-actif {
  background-color: #28a745;
}

.badge-inactif {
  background-color: #6c757d;
}

.btn-toggle {
  padding: 4px 8px;
  font-size: 12px;
  cursor: pointer;
  border: 1px solid #ccc;
  background-color: #f8f9fa;
  border-radius: 4px;
}

.btn-toggle:hover {
  background-color: #e2e6ea;
}

.vue-modifier {
  padding: 20px;
  background-color: #ffffff;
}

.vue-modifier div {
  margin-bottom: 10px;
}

.vue-modifier label {
  display: inline-block;
  width: 80px;
}

.vue-modifier input {
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.confirmation {
  padding: 20px;
  background-color: #ffffff;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.confirmation p {
  margin: 15px 0;
  font-size: 16px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.btn-plus {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 4px;
  cursor: pointer;
}

.btn-retour {
  margin-bottom: 15px;
  padding: 6px 12px;
  cursor: pointer;
}

table {
  width: 100%;
  border-collapse: collapse;
  background-color: #ffffff;
}

th, td {
  padding: 10px;
  text-align: left;
  border: 1px solid #dddddd;
}

th {
  background-color: #f8f9fa;
}

button {
  margin-right: 6px;
  padding: 5px 10px;
  cursor: pointer;
}

.btn-modifier {
  background-color: #ffc107;
  border: 1px solid #e0a800;
  border-radius: 3px;
}

.btn-supprimer {
  background-color: #dc3545;
  color: white;
  border: 1px solid #c82333;
  border-radius: 3px;
}

.btn-annuler {
  background-color: #6c757d;
  color: white;
  border: 1px solid #5a6268;
  border-radius: 3px;
}

.btn-valider {
  background-color: #28a745;
  color: white;
  border: 1px solid #218838;
  border-radius: 3px;
}
</style>
