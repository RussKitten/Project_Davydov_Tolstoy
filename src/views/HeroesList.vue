<template>
  <section class="heroes-page">
    <div class="page-header">
      <h1 class="page-title">Герои «Войны и мира»</h1>
      <p class="page-subtitle">Персонажи великого романа Льва Толстого</p>
    </div>

    <div class="search-section">
      <SearchBox v-model="q" placeholder="Поиск по героям, семьям, биографиям..." />
      <div class="search-stats" v-if="!loading && !error">
        Найдено {{ filtered.length }} из {{ heroes.length }} персонажей
      </div>
    </div>

    <hr class="sep" />
    
    <div v-if="loading" class="loading-state">
      <div class="spinner"></div>
      <p>Загрузка персонажей...</p>
    </div>
    
    <div v-else-if="error" class="error-state">
      <div class="error-icon">⚠️</div>
      <h3>Ошибка загрузки</h3>
      <p>{{ error }}</p>
      <button @click="loadAll" class="retry-btn">Попробовать снова</button>
    </div>
    
    <!-- Hero Cards Grid -->
    <div v-else class="heroes-container">
      <div class="heroes-grid" v-if="filtered.length">
        <div
          v-for="hero in filtered"
          :key="hero.id"
          class="hero-card"
          @click="openHeroPopup(hero)"
        >
          <div class="hero-avatar-container">
            <img 
              :src="`/img/heroes/${hero.id}.jpg`" 
              :alt="hero.name"
              class="hero-avatar"
              @error="onImgError"
            />
            <div class="hero-family-badge">{{ hero.family }}</div>
          </div>
          <div class="hero-info">
            <h3 class="hero-name">{{ hero.name }}</h3>
            <p class="hero-aliases" v-if="hero.aliases?.length">
              {{ hero.aliases.join(', ') }}
            </p>
            <p class="hero-bio-preview">{{ hero.smbio }}</p>
            <div class="hero-stats">
              <span v-if="hero.links?.relatives?.length" class="stat">
                👨‍👩‍👧‍👦 {{ hero.links.relatives.length }}
              </span>
              <span v-if="hero.links?.events?.length" class="stat">
                📅 {{ hero.links.events.length }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div v-else class="empty-state">
        <div class="empty-icon">🔍</div>
        <h3>Персонажи не найдены</h3>
        <p>Попробуйте изменить поисковый запрос</p>
        <button @click="q = ''" class="clear-search-btn">Очистить поиск</button>
      </div>
    </div>

    <!-- Hero Popup Modal -->
    <div v-if="selectedHero" class="popup-overlay" @click="closeAllPopups">
      <div class="hero-popup" @click.stop>
        <button class="close-btn" @click="closeAllPopups">×</button>
        
        <div class="popup-content">
          <div class="hero-main-info">
            <img 
              :src="`/img/heroes/${selectedHero.id}.jpg`" 
              :alt="selectedHero.name"
              class="hero-popup-img"
              @error="onImgError"
            />
            <div class="hero-basic-info">
              <h2>{{ selectedHero.name }}</h2>
              <p class="hero-family">{{ selectedHero.family }}</p>
              <p class="hero-sm-bio">{{ selectedHero.smbio }}</p>
              
              <div class="hero-actions">
                <button class="view-details-btn" @click="openFullDetails(selectedHero)">
                  📖 Полная биография
                </button>
              </div>
            </div>
          </div>

          <!-- Relatives Section -->
          <div v-if="selectedHero.links?.relatives?.length" class="relatives-section">
            <h3>Родственные связи</h3>
            <div class="relatives-grid">
              <div
                v-for="relativeId in selectedHero.links.relatives"
                :key="relativeId"
                class="relative-card"
                @click="openRelativePopup(relativeId)"
              >
                <img 
                  :src="`/img/heroes/${relativeId}.jpg`" 
                  :alt="idToName[relativeId]"
                  class="relative-avatar"
                  @error="onImgError"
                />
                <div class="relative-info">
                  <span class="relative-name">{{ idToName[relativeId] || relativeId }}</span>
                  <span class="relative-family">{{ getHeroFamily(relativeId) }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Relative Popup Modal -->
    <div v-if="selectedRelative" class="popup-overlay" @click="closeAllPopups">
      <div class="relative-popup" @click.stop>
        <button class="close-btn" @click="closeAllPopups">×</button>
        
        <div class="popup-content">
          <div class="hero-main-info">
            <img 
              :src="`/img/heroes/${selectedRelative.id}.jpg`" 
              :alt="selectedRelative.name"
              class="hero-popup-img"
              @error="onImgError"
            />
            <div class="hero-basic-info">
              <h2>{{ selectedRelative.name }}</h2>
              <p class="hero-family">{{ selectedRelative.family }}</p>
              <p class="hero-sm-bio">{{ selectedRelative.smbio }}</p>
              <button class="view-details-btn" @click="openFullDetails(selectedRelative)">
                📖 Полная биография
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Full Details Modal -->
    <div v-if="fullDetailsHero" class="popup-overlay full-details-overlay" @click="closeFullDetails">
      <div class="full-details-popup" @click.stop>
        <button class="close-btn" @click="closeFullDetails">×</button>
        
        <div class="full-details-content">
          <div class="details-header">
            <img 
              :src="`/img/heroes/${fullDetailsHero.id}.jpg`" 
              :alt="fullDetailsHero.name"
              class="details-hero-img"
              @error="onImgError"
            />
            <div class="details-title">
              <h1>{{ fullDetailsHero.name }}</h1>
              <p class="hero-family-large">{{ fullDetailsHero.family }}</p>
              <div v-if="fullDetailsHero.aliases?.length" class="aliases">
                <strong>Также известен как:</strong>
                <div class="aliases-list">
                  <span v-for="alias in fullDetailsHero.aliases" :key="alias" class="alias-tag">
                    {{ alias }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="details-body">
            <section class="bio-section">
              <h3>📖 Биография</h3>
              <p class="full-bio">{{ fullDetailsHero.bio }}</p>
            </section>

            <div class="details-grid">
              <section v-if="fullDetailsHero.links?.relatives?.length" class="relatives-details">
                <h3>👨‍👩‍👧‍👦 Родственники</h3>
                <div class="relatives-list">
                  <div
                    v-for="relativeId in fullDetailsHero.links.relatives"
                    :key="relativeId"
                    class="relative-item"
                    @click="openRelativeDetails(relativeId)"
                  >
                    <img 
                      :src="`/img/heroes/${relativeId}.jpg`" 
                      :alt="idToName[relativeId]"
                      class="relative-avatar-small"
                      @error="onImgError"
                    />
                    <div class="relative-details">
                      <span class="relative-name">{{ idToName[relativeId] || relativeId }}</span>
                      <span class="relative-family">{{ getHeroFamily(relativeId) }}</span>
                    </div>
                  </div>
                </div>
              </section>

              <section v-if="fullDetailsHero.links?.events?.length" class="events-section">
                <h3>📅 События с участием персонажа</h3>
                <div class="events-list">
                  <RouterLink
                    v-for="eventId in fullDetailsHero.links.events"
                    :key="eventId"
                    :to="`/events/${eventId}`"
                    class="event-link"
                  >
                    <span class="event-title">{{ eventsById[eventId]?.title || eventId }}</span>
                    <span class="event-date">{{ formatEventDate(eventsById[eventId]?.date) }}</span>
                  </RouterLink>
                </div>
              </section>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, onMounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useData } from '../composables/useData.js'
import SearchBox from '../components/SearchBox.vue'

const { loadAll, loading, error, heroes, eventsById } = useData()
const route = useRoute()
const router = useRouter()
const q = ref('')

// State management
const selectedHero = ref(null)
const selectedRelative = ref(null)
const fullDetailsHero = ref(null)

const idToName = computed(() => {
  if (!heroes.value) return {}
  const m = {}
  for (const h of heroes.value) m[h.id] = h.name
  return m
})

const getHeroFamily = (heroId) => {
  const hero = heroes.value?.find(h => h.id === heroId)
  return hero?.family || ''
}

const onImgError = (e) => {
  e.target.src = '/img/tolstoy.jpg'
}

const formatEventDate = (dateString) => {
  if (!dateString) return ''
  return new Date(dateString).toLocaleDateString('ru-RU', {
    year: 'numeric',
    month: 'short'
  })
}

// Hero popup functions
const scrollToPopup = () => {
  nextTick(() => {
    setTimeout(() => {
      const activePopup = document.querySelector('.hero-popup, .relative-popup, .full-details-popup')
      
      if (activePopup) {
        const rect = activePopup.getBoundingClientRect()
        const isMobile = window.innerWidth < 768
        const offset = isMobile ? 120 : 80 // Больший отступ для мобильных
        
        const targetScroll = window.pageYOffset + rect.top - offset
        
        window.scrollTo({
          top: Math.max(0, targetScroll), // Не уходим в отрицательные значения
          behavior: 'smooth'
        })
      }
    }, 50) // Небольшая задержка для гарантии рендера попапа
  })
}

const openHeroPopup = (hero) => {
  selectedHero.value = hero
  selectedRelative.value = null
  fullDetailsHero.value = null
  scrollToPopup()
}


const openRelativePopup = (relativeId) => {
  if (!heroes.value) return
  const relative = heroes.value.find(h => h.id === relativeId)
  if (relative) {
    selectedRelative.value = relative
    selectedHero.value = null
    fullDetailsHero.value = null
    scrollToPopup()
  }
}

const closeAllPopups = () => {
  selectedHero.value = null
  selectedRelative.value = null
}

// Full details functions
const openFullDetails = (hero) => {
  fullDetailsHero.value = hero
  selectedHero.value = null
  selectedRelative.value = null
  scrollToPopup()
}

const closeFullDetails = () => {
  fullDetailsHero.value = null
}

const openRelativeDetails = (relativeId) => {
  if (!heroes.value) return
  const relative = heroes.value.find(h => h.id === relativeId)
  if (relative) {
    openFullDetails(relative)
  }
}

const scrollToHeroEvents = (hero) => {
  // Implementation for scrolling to hero events
  console.log('Scroll to events for:', hero.name)
}

const filtered = computed(() => {
  if (!heroes.value) return []
  const t = q.value.trim().toLowerCase()
  if (!t) return heroes.value
  
  return heroes.value.filter(h => {
    const searchText = [
      h.name, 
      h.family, 
      h.bio, 
      h.smbio,
      ...(h.aliases || [])
    ].join(' ').toLowerCase()
    
    return searchText.includes(t)
  })
})
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

onMounted(() => {
  loadAll().then(() => {
    if (route.query.heroId && heroes.value) {
      const heroId = route.query.heroId
      const hero = heroes.value.find(h => h.id === heroId)
      if (hero) {
        openHeroPopup(hero)
      }
    }
  })
})

watch(
  () => route.query.heroId,
  (heroId) => {
    if (heroId && heroes.value) {
      const hero = heroes.value.find(h => h.id === heroId)
      if (hero) {
        openHeroPopup(hero)
      }
    }
  }
)
</script>

<style scoped>
.heroes-page {
  position: relative;
  min-height: 100vh;
  background: var(--background);
}

.page-header {
  text-align: center;
  padding: 40px 20px 20px;
}

.page-title {
  font-size: 2.5em;
  color: var(--peach);
  margin: 0 0 8px;
  font-weight: 700;
}

.page-subtitle {
  font-size: 1.1em;
  color: var(--text-secondary);
  margin: 0;
  opacity: 0.8;
}

.search-section {
  padding: 0 20px 20px;
  max-width: 600px;
  margin: 0 auto;
}

.search-stats {
  text-align: center;
  font-size: 0.9em;
  color: var(--text-secondary);
  margin-top: 8px;
}

.loading-state,
.error-state,
.empty-state {
  text-align: center;
  padding: 60px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid var(--line);
  border-top: 4px solid var(--peach);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-icon,
.empty-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.retry-btn,
.clear-search-btn {
  padding: 10px 20px;
  background: var(--peach);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
}

.retry-btn:hover,
.clear-search-btn:hover {
  background: #a67c52;
  transform: translateY(-2px);
}

.heroes-container {
  padding: 20px;
}

.heroes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 24px;
  max-width: 1400px;
  margin: 0 auto;
}

.hero-card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 16px;
  padding: 0;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.hero-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  border-color: var(--peach);
}

.hero-avatar-container {
  position: relative;
  height: 200px;
  overflow: hidden;
}

.hero-avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.hero-card:hover .hero-avatar {
  transform: scale(1.05);
}

.hero-family-badge {
  position: absolute;
  top: 12px;
  left: 12px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 0.8em;
  font-weight: 500;
}

.hero-info {
  padding: 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.hero-name {
  margin: 0 0 8px;
  font-size: 1.3em;
  color: var(--peach);
  line-height: 1.3;
}

.hero-aliases {
  font-size: 0.9em;
  color: var(--text-secondary);
  margin: 0 0 12px;
  font-style: italic;
}

.hero-bio-preview {
  flex: 1;
  line-height: 1.5;
  margin: 0 0 16px;
  color: var(--text);
}

.hero-stats {
  display: flex;
  gap: 12px;
}

.stat {
  font-size: 0.8em;
  color: var(--text-secondary);
  background: var(--background);
  padding: 4px 8px;
  border-radius: 6px;
}

/* Popup Styles */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
  backdrop-filter: blur(4px);
}

.hero-popup,
.relative-popup {
  background: var(--card);
  border-radius: 20px;
  padding: 30px;
  max-width: 500px;
  width: 100%;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
  border: 2px solid var(--peach);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  font-size: 24px;
  color: var(--peach);
  cursor: pointer;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  z-index: 1;
}

.close-btn:hover {
  background: rgba(187, 148, 87, 0.1);
}

.hero-main-info {
  display: flex;
  gap: 24px;
  align-items: flex-start;
  margin-bottom: 24px;
}

.hero-popup-img {
  width: 120px;
  height: 120px;
  border-radius: 12px;
  object-fit: cover;
  flex-shrink: 0;
  border: 3px solid var(--peach);
}

.hero-basic-info {
  flex: 1;
}

.hero-basic-info h2 {
  margin: 0 0 8px;
  color: var(--peach);
  font-size: 1.5em;
}

.hero-actions {
  display: flex;
  gap: 12px;
  margin-top: 16px;
  flex-wrap: wrap;
}

.view-details-btn,
.events-btn {
  padding: 8px 16px;
  background: linear-gradient(180deg, rgba(187, 148, 87, 0.25), rgba(153, 88, 42, 0.25));
  border: 1px solid rgba(255, 230, 167, 0.35);
  color: var(--peach);
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9em;
  transition: all 0.3s ease;
}

.view-details-btn:hover,
.events-btn:hover {
  background: linear-gradient(180deg, rgba(187, 148, 87, 0.4), rgba(153, 88, 42, 0.4));
  transform: translateY(-2px);
}

.relatives-section h3 {
  margin: 0 0 16px;
  color: var(--peach);
  font-size: 1.2em;
}

.relatives-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 12px;
}

.relative-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid var(--line);
}

.relative-card:hover {
  background: rgba(187, 148, 87, 0.1);
  border-color: var(--peach);
  transform: translateY(-2px);
}

.relative-avatar {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.relative-info {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
}

.relative-name {
  font-weight: 600;
  font-size: 0.9em;
  margin-bottom: 2px;
}

.relative-family {
  font-size: 0.8em;
  color: var(--text-secondary);
}

/* Full Details Popup */
.full-details-overlay {
  align-items: flex-start;
  padding-top: 40px;
  padding-bottom: 40px;
}

.full-details-popup {
  background: var(--card);
  border-radius: 20px;
  padding: 30px;
  max-width: 900px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  border: 2px solid var(--peach);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
}

.details-header {
  display: flex;
  gap: 30px;
  margin-bottom: 30px;
  align-items: flex-start;
}

.details-hero-img {
  width: 150px;
  height: 150px;
  border-radius: 12px;
  object-fit: cover;
  flex-shrink: 0;
  border: 3px solid var(--peach);
}

.details-title h1 {
  margin: 0 0 8px;
  color: var(--peach);
  font-size: 2em;
}

.hero-family-large {
  font-size: 1.2em;
  color: var(--text-secondary);
  margin: 0 0 16px;
}

.aliases {
  margin-top: 16px;
}

.aliases strong {
  display: block;
  margin-bottom: 8px;
  color: var(--text-secondary);
}

.aliases-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.alias-tag {
  display: inline-block;
  background: rgba(187, 148, 87, 0.2);
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.9em;
  border: 1px solid rgba(187, 148, 87, 0.3);
}

.details-body {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.bio-section h3,
.relatives-details h3,
.events-section h3 {
  margin: 0 0 16px;
  color: var(--peach);
  font-size: 1.3em;
  display: flex;
  align-items: center;
  gap: 8px;
}

.full-bio {
  line-height: 1.7;
  margin: 0;
  font-size: 1.05em;
}

.details-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.relatives-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.relative-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid var(--line);
}

.relative-item:hover {
  background: rgba(187, 148, 87, 0.1);
  border-color: var(--peach);
  transform: translateX(4px);
}

.relative-avatar-small {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.relative-details {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.events-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.event-link {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: var(--peach);
  text-decoration: none;
  padding: 12px;
  border-radius: 8px;
  transition: all 0.3s ease;
  border: 1px solid var(--line);
}

.event-link:hover {
  background: rgba(187, 148, 87, 0.1);
  border-color: var(--peach);
  text-decoration: none;
  transform: translateX(4px);
}

.event-title {
  font-weight: 500;
  flex: 1;
}

.event-date {
  font-size: 0.9em;
  color: var(--text-secondary);
  margin-left: 12px;
}

/* Responsive Design */
@media (max-width: 768px) {
  .page-title {
    font-size: 2em;
  }

  .heroes-grid {
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 16px;
  }

  .hero-main-info {
    flex-direction: column;
    text-align: center;
  }

  .hero-popup-img {
    align-self: center;
  }

  .details-header {
    flex-direction: column;
    text-align: center;
  }

  .details-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }

  .full-details-popup {
    margin: 20px;
    padding: 20px;
  }

  .relatives-grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }

  .hero-actions {
    flex-direction: column;
  }

  .event-link {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .event-date {
    margin-left: 0;
  }
}

@media (max-width: 480px) {
  .heroes-grid {
    grid-template-columns: 1fr;
  }

  .hero-popup,
  .relative-popup {
    margin: 10px;
    padding: 20px;
  }

  .page-header {
    padding: 20px 16px 16px;
  }

  .heroes-container {
    padding: 16px;
  }
}
</style>
