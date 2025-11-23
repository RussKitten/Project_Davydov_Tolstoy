<template>
  <section>
    <h2 class="section-title">Таймлайн</h2>
    <div v-if="loading">Загрузка…</div>
    <div v-else-if="error" class="small">Ошибка: {{ error }}</div>
    <div v-else>
      <ul class="clean">
        <li v-for="ev in sorted" :key="ev.id" class="card">
          <div class="small mono">{{ ev.date }}</div>
          <h3 style="margin:6px 0 6px;">
            <RouterLink :to="`/events/${ev.id}`">{{ ev.title }}</RouterLink>
          </h3>
          <div class="small">
            {{ locationsById[ev.placeId]?.name || '—' }}
          </div>
          <p>{{ ev.summary }}</p>
        </li>
      </ul>
    </div>
  </section>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useData } from '../composables/useData.js'

const { loadAll, loading, error, events, locationsById } = useData()
onMounted(loadAll)

const sorted = computed(() => [...events.value].sort((a,b) => (a.date||'').localeCompare(b.date||'')))
</script>