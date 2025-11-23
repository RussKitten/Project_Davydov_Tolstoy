<template>
  <section>
    <h2 class="section-title">События</h2>
    <SearchBox v-model="q" placeholder="Поиск по событиям…" />
    <hr class="sep" />
    <div v-if="loading">Загрузка…</div>
    <div v-else-if="error" class="small">Ошибка: {{ error }}</div>
    <div class="grid" v-else>
      <article v-for="ev in filtered" :key="ev.id" class="card">
        <h3><RouterLink :to="`/events/${ev.id}`">{{ ev.title }}</RouterLink></h3>
        <p class="small mono">{{ ev.date }} • {{ locationsById[ev.placeId]?.name || '—' }}</p>
        <p>{{ ev.summary }}</p>
        <div class="row small">
          <span class="tag" v-for="pid in ev.participants" :key="pid">{{ heroesById[pid]?.name || pid }}</span>
        </div>
      </article>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, onMounted } from 'vue'
import { useData } from '../composables/useData.js'
import SearchBox from '../components/SearchBox.vue'

const { loadAll, loading, error, events, heroesById, locationsById } = useData()
const q = ref('')

onMounted(loadAll)

const filtered = computed(() => {
  const t = q.value.trim().toLowerCase()
  if (!t) return sorted(events.value)
  return sorted(events.value).filter(e =>
    [e.title, e.summary, e.date, (locationsById.value[e.placeId]?.name || '')]
      .join(' ')
      .toLowerCase()
      .includes(t)
  )
})

function sorted(arr) {
  return [...arr].sort((a, b) => (a.date || '').localeCompare(b.date || ''))
}
</script>