<template>
  <section>
    <h2 class="section-title">Темы</h2>
    <div v-if="loading">Загрузка…</div>
    <div v-else-if="error" class="small">Ошибка: {{ error }}</div>
    <div class="grid" v-else>
      <article v-for="th in themes" :key="th.id" class="card">
        <h3><RouterLink :to="`/themes/${th.id}`">{{ th.title }}</RouterLink></h3>
        <p>{{ th.summary }}</p>
        <div class="row small">
          <span class="tag" v-for="pid in (th.heroes || [])" :key="pid">{{ heroesById[pid]?.name || pid }}</span>
        </div>
      </article>
    </div>
  </section>
</template>

<script setup>
import { onMounted } from 'vue'
import { useData } from '../composables/useData.js'

const { loadAll, loading, error, themes, heroesById } = useData()
onMounted(loadAll)
</script>