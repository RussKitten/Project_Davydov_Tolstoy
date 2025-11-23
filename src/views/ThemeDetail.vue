<template>
  <section v-if="th" class="card">
    <h2 class="section-title">{{ th.title }}</h2>
    <p>{{ th.summary }}</p>

    <div v-if="th.heroes?.length">
      <h3>Ключевые персонажи</h3>
      <div class="row">
        <RouterLink v-for="pid in th.heroes" :key="pid" class="tag" :to="`/heroes/${pid}`">
          {{ heroesById[pid]?.name || pid }}
        </RouterLink>
      </div>
    </div>

    <div v-if="th.events?.length">
      <h3>Примеры эпизодов</h3>
      <ul class="clean">
        <li v-for="eid in th.events" :key="eid">
          <RouterLink :to="`/events/${eid}`">{{ eventsById[eid]?.title || eid }}</RouterLink>
          <span class="small mono" v-if="eventsById[eid]?.date"> — {{ eventsById[eid]?.date }}</span>
        </li>
      </ul>
    </div>
  </section>
  <section v-else class="small">Тема не найдена.</section>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { useData } from '../composables/useData.js'

const route = useRoute()
const { loadAll, themesById, heroesById, eventsById } = useData()

onMounted(loadAll)

const th = computed(() => themesById.value[route.params.id])
</script>