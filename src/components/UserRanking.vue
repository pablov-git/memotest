<script setup>
import { ref, onMounted } from 'vue'

const ranking = ref([])

const props = defineProps({
  difficulty: { type: String},
  currentPlayer: { type: Object }
})

onMounted(() => {
  const stored = JSON.parse(localStorage.getItem('ranking' + props.difficulty) || '[]')
  ranking.value = stored
})
</script>

<template>
  <div class="ranking-container">
    <h3>🏆 Ranking - Mode: {{ difficulty }} </h3>
    <table v-if="ranking.length > 0">
      <thead>
        <tr>
          <th>#</th>
          <th>Nombre</th>
          <th>Errores</th>
          <th>Tiempo</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, index) in ranking" :key="index"  :class="{ 'current-player': user.name === currentPlayer?.name && user.errors === currentPlayer?.errors && user.time === currentPlayer?.time }">
          <td>{{ index + 1 }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.errors }}</td>
          <td>{{ user.time }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>No hay puntuaciones aún.</p>
  </div>
</template>

<style scoped>
.ranking-container {
  margin-top: 2rem;
  padding: 1rem;
  border-radius: 12px;
  background-color: #f8f9fa;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  text-align: center;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

th, td {
  padding: 0.6rem 1rem;
  border-bottom: 1px solid #ccc;
}

th {
  background-color: var(--sea-blue);
  color: white;
}

tbody tr:nth-child(even) {
  background-color: #eef2f3;
}

.current-player td {
  font-weight: bold;
}
</style>
