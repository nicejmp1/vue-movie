<template>
  <HeaderSection />
  <section>
    <div class="search-container">
      <h2>검색 결과</h2>
      <div v-if="searchResults.length > 0" class="search-results">
        <ul>
          <li v-for="movie in searchResults" :key="movie.id" class="search-item">
            <router-link :to="{ name: 'detail', params: { movieID: movie.id } }">
              <img :src="getMovieImage(movie.poster_path)" alt="movie.title" class="poster" />
              <div class="movie-info">
                <h3>{{ movie.title }}</h3>
                <p>{{ movie.release_date }}</p>
                <p class="rating">
                  <svg-icon type="mdi" :path="mdiStar"></svg-icon>{{ movie.vote_average }}
                </p>
              </div>
            </router-link>
          </li>
        </ul>
      </div>
      <div v-else>검색 결과가 없습니다.</div>
    </div>
  </section>
  <FooterSection />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'
import SvgIcon from '@jamescoyle/vue-icon'
import { mdiStar } from '@mdi/js'

const route = useRoute()
const query = ref(route.query.q || '')
const searchResults = ref([])

const searchMovies = async () => {
  if (query.value.trim() === '') {
    searchResults.value = []
    return
  }
  try {
    const response = await axios.get(`https://api.themoviedb.org/3/search/movie`, {
      params: {
        api_key: import.meta.env.VITE_TMDB_API_KEY,
        language: 'ko',
        query: query.value
      }
    })
    searchResults.value = response.data.results
  } catch (error) {
    console.error('영화 검색 중 오류 발생:', error)
  }
}

const getMovieImage = (path) => {
  return path ? `https://image.tmdb.org/t/p/w200/${path}` : 'path/to/default-image.jpg'
}

onMounted(searchMovies)
watch(
  () => route.query.q,
  (newQuery) => {
    query.value = newQuery
    searchMovies()
  }
)
</script>

<style scoped>
.search-container {
  padding: 20px;
  max-width: 800px;
  margin: 0 auto;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
h2 {
  font-family: var(--fontN);
  font-weight: bold;
  font-size: 30px;
  margin-bottom: 20px;
  text-align: center;
  color: #333;
}
.search-results ul {
  list-style-type: none;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 20px;
}
.search-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  background-color: #fff;
  padding: 10px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
}
.search-item:hover {
  transform: translateY(-10px);
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}
.poster {
  width: 100%;
  height: auto;
  border-radius: 10px;
  margin-bottom: 10px;
}
.movie-info {
  h3 {
    margin: 0;
    font-size: 1.2rem;
    color: #333;
  }
  p {
    margin: 5px 0;
    color: #777;
  }
  .rating {
    font-weight: bold;
    color: #f39c12;

    svg {
      vertical-align: -6px;
      margin-right: 4px;
    }
  }
}
</style>
