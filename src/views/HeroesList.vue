<template>
  <section>
    <h2 class="section-title">Герои</h2>
    <SearchBox v-model="q" placeholder="Поиск по героям…" />
    <hr class="sep" />
    <div v-if="loading">Загрузка…</div>
    <div v-else-if="error" class="small">Ошибка: {{ error }}</div>
    <div class="grid" v-else>
      <article
        v-for="ch in filtered"
        :key="ch.id"
        :ref="el => el && (cardRefs[ch.id] = el)"
        :class="['card', { expanded: expandedIds.has(ch.id) }]"
      >
        <h3>
          <a href="#" @click.prevent="toggle(ch.id)">{{ ch.name }}</a>
        </h3>
        <p class="small mono">{{ ch.family }}</p>
        <p>{{ ch.smbio }}</p>
        <div class="row" v-if="ch.aliases?.length">
          <span class="tag" v-for="a in ch.aliases" :key="a">Также: {{ a }}</span>
        </div>
        <div v-if="expandedIds.has(ch.id)" class="expanded-content">
          <img class="portrait" :src="`/img/heroes/${ch.id}.jpg`" :alt="ch.name" @error="onImgError" />
          <div class="details">
            <p class="bio">{{ ch.bio }}</p>
            <div v-if="ch.links?.relatives?.length" class="row wrap">
              <span class="label">Родственники:</span>
              <RouterLink v-for="rid in ch.links.relatives" :key="rid" class="tag" :to="`/heroes/${rid}`">{{ idToName[rid] || rid }}</RouterLink>
            </div>
            <div v-if="ch.links?.events?.length" class="row wrap">
              <span class="label">События:</span>
              <RouterLink v-for="eid in ch.links.events" :key="eid" class="tag" :to="`/events/${eid}`">{{ eventsById[eid]?.title || eid }}</RouterLink>
            </div>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, onMounted, nextTick, reactive } from 'vue'
import { useData } from '../composables/useData.js'
import SearchBox from '../components/SearchBox.vue'

const { loadAll, loading, error, heroes, eventsById } = useData()
const q = ref('')

// track up to three expanded cards
const expandedIds = reactive(new Set())
const cardRefs = reactive({})
const toggle = async (id) => {
  if (expandedIds.has(id)) {
    expandedIds.delete(id)
    return
  }
  // add; if exceeds 3, remove the oldest (first inserted)
  expandedIds.add(id)
  if (expandedIds.size > 3) {
    const first = expandedIds.values().next().value
    expandedIds.delete(first)
  }
  await nextTick()
  const el = cardRefs[id]
  if (el && typeof el.scrollIntoView === 'function') {
    el.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}
const idToName = computed(() => {
  const m = {}
  for (const h of heroes.value) m[h.id] = h.name
  return m
})
const onImgError = (e) => {
  e.target.src = '/img/tolstoy.jpg'
}

onMounted(loadAll)

const filtered = computed(() => {
  const t = q.value.trim().toLowerCase()
  if (!t) return heroes.value
  return heroes.value.filter(h =>
    [h.name, h.family, h.bio, ...(h.aliases || [])]
      .join(' ')
      .toLowerCase()
      .includes(t)
  )
})
</script>
<style scoped>
.card h3 {
  display: flex;
  align-items: baseline;
  gap: 12px;
}
.card h3 a {
  color: inherit;
  text-decoration: none;
  cursor: pointer;
}
.card h3 a:hover {
  text-decoration: underline;
}
.more-link {
  font-size: 0.8rem;
  opacity: 0.7;
}
.card.expanded {
  grid-column: 1 / -1;
  scroll-margin-top: 16px;
}
.expanded-content {
  display: grid;
  grid-template-columns: 150px 1fr;
  gap: 16px;
  margin-top: 12px;
  align-items: start;
}
.portrait {
  width: 150px;
  height: 150px;
  object-fit: cover;
  border-radius: 8px;
  background: #f1f1f1;
}
.details .row.wrap {
  flex-wrap: wrap;
  gap: 8px;
}
.label {
  font-weight: 600;
  margin-right: 8px;
}
.tag {
  margin-top: 4px;
}
</style>