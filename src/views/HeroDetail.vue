<template>
  <section v-if="hero" class="card">
    <h2 class="section-title">{{ hero.name }}</h2>
    <p class="small mono">{{ hero.family }}</p>
    <p>{{ hero.bio }}</p>

    <div v-if="hero.links?.relatives?.length">
      <h3>Связанные персонажи</h3>
      <div class="row">
        <RouterLink v-for="rid in hero.links.relatives" :key="rid" class="tag" :to="`/heroes/${rid}`">
          {{ heroesById[rid]?.name || rid }}
        </RouterLink>
      </div>
    </div>

    <div v-if="hero.links?.events?.length">
      <h3>Ключевые эпизоды</h3>
      <ul class="clean">
        <li v-for="eid in hero.links.events" :key="eid">
          <RouterLink :to="`/events/${eid}`">{{ eventsById[eid]?.title || eid }}</RouterLink>
          <span class="small mono" v-if="eventsById[eid]?.date"> — {{ eventsById[eid]?.date }}</span>
        </li>
      </ul>
    </div>
  </section>
  <section v-else class="small">Персонаж не найден.</section>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { useData } from '../composables/useData.js'

const route = useRoute()
const { loadAll, heroesById, eventsById } = useData()

onMounted(loadAll)

const hero = computed(() => heroesById.value[route.params.id])
</script>