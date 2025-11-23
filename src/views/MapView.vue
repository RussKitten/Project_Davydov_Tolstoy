<template>
  <section>
    <h2 class="section-title">Карта локаций</h2>
    <p class="small">Нажмите на ссылку, чтобы открыть локацию в OpenStreetMap.</p>
    <div v-if="loading">Загрузка…</div>
    <div v-else-if="error" class="small">Ошибка: {{ error }}</div>
    <div class="grid" v-else>
      <article v-for="loc in locations" :key="loc.id" class="card">
        <h3>{{ loc.name }}</h3>
        <p class="small mono">{{ loc.type }} • {{ loc.lat }}, {{ loc.lng }}</p>
        <p>
          <a :href="mapUrl(loc)" target="_blank" rel="noopener">Открыть на карте</a>
        </p>
        <div v-if="eventsAt(loc.id)?.length">
          <h4 style="margin:10px 0 6px;">Связанные события</h4>
          <ul class="clean small">
            <li v-for="ev in eventsAt(loc.id)" :key="ev.id">
              <RouterLink :to="`/events/${ev.id}`">{{ ev.title }}</RouterLink> — {{ ev.date }}
            </li>
          </ul>
        </div>
      </article>
    </div>
  </section>
</template>

<script setup>
import { onMounted } from 'vue'
import { useData } from '../composables/useData.js'

const { loadAll, loading, error, locations, events } = useData()
onMounted(loadAll)

function mapUrl(loc) {
  const z = 9
  return `https://www.openstreetmap.org/?mlat=${loc.lat}&mlon=${loc.lng}#map=${z}/${loc.lat}/${loc.lng}`
}

function eventsAt(placeId) {
  return events.value.filter(e => e.placeId === placeId)
}
</script>