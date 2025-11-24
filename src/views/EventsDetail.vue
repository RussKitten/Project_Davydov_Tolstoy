<template>
  <div class="events-detail-page">
    <div class="back-card">
      <RouterLink to="/events" class="back-link">
        <span class="back-arrow">←</span>
        Назад к событиям
      </RouterLink>
    </div>
    
    <section v-if="ev" class="card event-card">
      <div class="event-header">
        <h1 class="event-title">{{ ev.title }}</h1>
        <div class="event-meta">
          <span class="event-date">{{ formatDate(ev.date) }}</span>
          <span class="event-location" v-if="loc">
            • {{ loc.name }} <span class="location-type">({{ formatLocationType(loc.type) }})</span>
          </span>
          <span class="event-type-badge" :class="ev.type">{{ formatEventType(ev.type) }}</span>
        </div>
      </div>

      <!-- Участники события -->
      <div v-if="ev.participants?.length" class="participants-section">
        <h3 class="section-subtitle">Участники события</h3>
        <div class="participants-grid">
          <div
            v-for="pid in ev.participants"
            :key="pid"
            class="participant-card"
            @click="navigateToHero(pid)"
          >
            <img 
              :src="`/img/heroes/${pid}.jpg`" 
              :alt="heroesById[pid]?.name"
              class="participant-avatar"
              @error="onImgError"
            />
            <span class="participant-name">{{ heroesById[pid]?.name || pid }}</span>
            <span class="participant-family">{{ heroesById[pid]?.family }}</span>
          </div>
        </div>
      </div>

      <!-- Краткое содержание -->
      <div class="summary-section">
        <h2 class="section-subtitle">Краткое содержание</h2>
        <p class="event-summary">{{ ev.summary }}</p>
      </div>

      <!-- Значимость события -->
      <div v-if="ev.significance" class="significance-section">
        <h2 class="section-subtitle">Историческая значимость</h2>
        <p class="event-significance">{{ ev.significance }}</p>
      </div>

      <!-- Цитаты -->
      <div v-if="ev.quotes?.length" class="quotes-section">
        <h2 class="section-subtitle">Цитаты</h2>
        <div class="quotes-list">
          <blockquote 
            v-for="(quote, index) in ev.quotes" 
            :key="index" 
            class="quote-item"
          >
            "{{ quote }}"
          </blockquote>
        </div>
      </div>

      <!-- Изображение события -->
      <div class="event-image-section">
        <img 
          class="event-image" 
          :src="`/img/events/${ev.id}.jpg`" 
          :alt="ev.title"
          @error="onImgError" 
        />
      </div>

      <!-- Связанные события -->
      <div v-if="relatedEvents.length" class="related-events-section">
        <h2 class="section-subtitle">Связанные события</h2>
        <div class="related-events-grid">
          <RouterLink
            v-for="relatedEvent in relatedEvents"
            :key="relatedEvent.id"
            :to="`/events/${relatedEvent.id}`"
            class="related-event-card"
          >
            <h4>{{ relatedEvent.title }}</h4>
            <p class="related-event-date">{{ formatDate(relatedEvent.date) }}</p>
          </RouterLink>
        </div>
      </div>
    </section>

    <section v-else class="not-found-section">
      <div class="not-found-card">
        <h2>Событие не найдено</h2>
        <p>Запрошенное событие не существует или было удалено.</p>
        <RouterLink to="/events" class="back-to-events-btn">
          Вернуться к списку событий
        </RouterLink>
      </div>
    </section>
  </div>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useData } from '../composables/useData.js'

const props = defineProps({
  id: {
    type: String,
    required: true
  }
})

const route = useRoute()
const router = useRouter()
const { loadAll, eventsById, heroesById, locationsById, events } = useData()

onMounted(loadAll)

const ev = computed(() => eventsById.value[props.id])
const loc = computed(() => ev.value ? locationsById.value[ev.value.placeId] : null)

// Похожие события (по типу или участникам)
const relatedEvents = computed(() => {
  if (!ev.value || !events.value) return []
  
  return events.value
    .filter(event => 
      event.id !== ev.value.id && 
      (event.type === ev.value.type || 
       event.participants?.some(p => ev.value.participants?.includes(p)))
    )
    .slice(0, 3) // Ограничиваем 3 событиями
})

const navigateToHero = (heroId) => {
  router.push({
    path: '/heroes',
    query: { heroId }
  })
}

const onImgError = (e) => {
  e.target.src = '/img/tolstoy.jpg'
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('ru-RU', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const formatEventType = (type) => {
  const types = {
    'salon': 'Светский вечер',
    'battle': 'Сражение',
    'disaster': 'Катастрофа',
    'personal': 'Личное',
    'social': 'Социальное',
    'ball': 'Бал',
    'military': 'Военное',
    'war': 'Война',
    'scandal': 'Скандал',
    'duel': 'Дуэль'
  }
  return types[type] || type
}

const formatLocationType = (type) => {
  const types = {
    'city': 'Город',
    'battlefield': 'Поле боя',
    'estate': 'Имение',
    'palace': 'Дворец'
  }
  return types[type] || type
}
</script>

<style scoped>
.events-detail-page {
  min-height: 100vh;
  background: var(--background);
  padding-bottom: 40px;
}

.back-card {
  margin: 16px;
}

.back-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 12px;
  color: var(--peach);
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.back-link:hover {
  background: rgba(187, 148, 87, 0.1);
  border-color: var(--peach);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.back-arrow {
  font-size: 18px;
}

.card {
  border: 1px solid var(--line);
  padding: 24px;
  border-radius: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.25);
  background: var(--card);
  margin: 16px;
}

.event-header {
  margin-bottom: 24px;
  padding-bottom: 20px;
  border-bottom: 1px solid var(--line);
}

.event-title {
  margin: 0 0 12px;
  font-size: 28px;
  color: var(--peach);
  line-height: 1.3;
}

.event-meta {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  font-size: 14px;
  color: var(--text-secondary);
}

.event-type-badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.event-type-badge.salon,
.event-type-badge.social,
.event-type-badge.ball {
  background: rgba(101, 163, 13, 0.2);
  color: #65a30d;
  border: 1px solid rgba(101, 163, 13, 0.3);
}

.event-type-badge.battle,
.event-type-badge.war,
.event-type-badge.military {
  background: rgba(220, 38, 38, 0.2);
  color: #dc2626;
  border: 1px solid rgba(220, 38, 38, 0.3);
}

.event-type-badge.personal,
.event-type-badge.scandal {
  background: rgba(168, 85, 247, 0.2);
  color: #a855f7;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.event-type-badge.disaster {
  background: rgba(245, 158, 11, 0.2);
  color: #f59e0b;
  border: 1px solid rgba(245, 158, 11, 0.3);
}

.event-type-badge.duel {
  background: rgba(59, 130, 246, 0.2);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.section-subtitle {
  margin: 0 0 16px;
  font-size: 20px;
  color: var(--peach);
  font-weight: 600;
}

/* Участники */
.participants-section {
  margin: 32px 0;
}

.participants-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 16px;
}

.participant-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
  border: 1px solid var(--line);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  background: var(--card);
  text-align: center;
}

.participant-card:hover {
  border-color: var(--peach);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.participant-avatar {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 8px;
  border: 2px solid var(--peach);
}

.participant-name {
  font-weight: 600;
  color: var(--text);
  margin-bottom: 4px;
}

.participant-family {
  font-size: 12px;
  color: var(--text-secondary);
}

/* Содержание и значимость */
.summary-section,
.significance-section {
  margin: 32px 0;
}

.event-summary,
.event-significance {
  line-height: 1.7;
  font-size: 16px;
  color: var(--text);
  margin: 0;
}

/* Цитаты */
.quotes-section {
  margin: 32px 0;
}

.quotes-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.quote-item {
  padding: 20px;
  background: rgba(187, 148, 87, 0.05);
  border-left: 4px solid var(--peach);
  border-radius: 8px;
  font-style: italic;
  line-height: 1.6;
  margin: 0;
}

/* Изображение */
.event-image-section {
  margin: 32px 0;
  text-align: center;
}

.event-image {
  width: 100%;
  max-width: 500px;
  height: auto;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}

/* Связанные события */
.related-events-section {
  margin: 32px 0 0;
  padding-top: 24px;
  border-top: 1px solid var(--line);
}

.related-events-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 16px;
}

.related-event-card {
  padding: 16px;
  border: 1px solid var(--line);
  border-radius: 12px;
  text-decoration: none;
  color: inherit;
  transition: all 0.3s ease;
  background: var(--card);
}

.related-event-card:hover {
  border-color: var(--peach);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.related-event-card h4 {
  margin: 0 0 8px;
  color: var(--peach);
  font-size: 16px;
}

.related-event-date {
  font-size: 14px;
  color: var(--text-secondary);
  margin: 0;
}

/* Страница не найдена */
.not-found-section {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 60vh;
  padding: 20px;
}

.not-found-card {
  text-align: center;
  padding: 40px;
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 16px;
  max-width: 400px;
}

.not-found-card h2 {
  color: var(--peach);
  margin-bottom: 16px;
}

.not-found-card p {
  color: var(--text-secondary);
  margin-bottom: 24px;
}

.back-to-events-btn {
  display: inline-block;
  padding: 12px 24px;
  background: var(--peach);
  color: white;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.back-to-events-btn:hover {
  background: #a67c52;
  transform: translateY(-2px);
}

@media (min-width: 768px) {
  .card {
    margin: 24px auto;
    max-width: 800px;
    padding: 32px;
  }

  .back-card {
    margin: 24px auto;
    max-width: 800px;
  }

  .event-title {
    font-size: 32px;
  }

  .participants-grid {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 20px;
  }

  .event-image {
    max-width: 600px;
  }
}

@media (min-width: 1024px) {
  .card {
    max-width: 900px;
  }
  
  .back-card {
    max-width: 900px;
  }
}
</style>
