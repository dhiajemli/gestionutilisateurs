<template>
  <div class="formulaire">
    <h2>Ajouter un utilisateur</h2>

    <div>
      <label>ID : </label>
      <input type="text" v-model="id" />
    </div>

    <div>
      <label>Nom : </label>
      <input type="text" v-model="nom" />
    </div>

    <div>
      <label>Prénom : </label>
      <input type="text" v-model="prenom" />
    </div>

    <button @click="ajouter">Ajouter</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits(['utilisateur-ajoute'])

const id = ref('')
const nom = ref('')
const prenom = ref('')

function ajouter() {
  fetch('http://localhost:3000/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: Number(id.value) || id.value,
      nom: nom.value,
      prenom: prenom.value,
      actif: true
    })
  }).then(() => {
    id.value = ''
    nom.value = ''
    prenom.value = ''
    emit('utilisateur-ajoute')
  })
}
</script>

<style scoped>
.formulaire {
  padding: 20px;
  background-color: #ffffff;
  color: #333333;
}
div {
  margin-bottom: 10px;
}
label {
  display: inline-block;
  width: 80px;
}
input {
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  padding: 6px 14px;
  cursor: pointer;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
}
</style>
