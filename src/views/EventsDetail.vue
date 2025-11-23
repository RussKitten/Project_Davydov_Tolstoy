<template>
  <section v-if="ev" class="card">
    <h2 class="section-title">{{ ev.title }}</h2>
    <p class="small mono">
      {{ ev.date }} • {{ loc?.name }} <span v-if="loc">({{ loc.type }})</span>
    </p>
    <p>{{ ev.summary }}</p>
    <div v-if="ev.participants?.length">
      <h3>Участники</h3>
      <div class="row">
        <RouterLink v-for="pid in ev.participants" :key="pid" class="tag" :to="`/heroes/${pid}`">
          {{ heroesById[pid]?.name || pid }}
        </RouterLink>
      </div>
    </div>
  </section>
  <section v-else class="small">Событие не найдено.</section>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { useData } from '../composables/useData.js'

const route = useRoute()
const { loadAll, eventsById, heroesById, locationsById } = useData()

onMounted(loadAll)

const ev = computed(() => eventsById.value[route.params.id])
const loc = computed(() => ev.value ? locationsById.value[ev.value.placeId] : null)
</script>