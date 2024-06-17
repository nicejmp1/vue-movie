<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'
import Modal from '@/components/ModalSection.vue'

const latestMovies = ref([])
const popularMovies = ref([])
const movieVideo = ref(null)
const modalRef = ref(null)
const apikey = import.meta.env.VITE_TMDB_API_KEY

const fetchMovies = async () => {
  try {
    const latestResponse = await axios.get(
      `https://api.themoviedb.org/3/movie/now_playing?api_key=${apikey}&language=ko&page=1`
    )
    latestMovies.value = latestResponse.data.results
    console.log(latestResponse)
  } catch (error) {
    console.error(error)
  }
}

const fetchPopular = async () => {
  try {
    const popularResponse = await axios.get(
      `https://api.themoviedb.org/3/movie/popular?api_key=${apikey}&page=1`
    )
    popularMovies.value = popularResponse.data.results
    console.log(popularResponse)
  } catch (error) {
    console.error(error)
  }
}

const fetchVideo = async (movie_id) => {
  try {
    const videoResponse = await axios.get(
      `https://api.themoviedb.org/3/movie/${movie_id}/videos?api_key=${apikey}&language=ko`
    )
    const videos = videoResponse.data.results
    if (videos.length > 0) {
      movieVideo.value = videos[0] // 첫 번째 비디오만 저장
      modalRef.value.showModal() // 모달을 표시
    } else {
      movieVideo.value = null // 비디오가 없으면 null로 설정
    }
    console.log(videoResponse)
  } catch (error) {
    console.error(error)
    movieVideo.value = null // 오류 발생 시 null로 설정
  }
}

onMounted(fetchMovies)
onMounted(fetchPopular)
</script>

<script>
import { Swiper, SwiperSlide } from 'swiper/vue'

// Import Swiper styles
import 'swiper/css'
import 'swiper/css/pagination'
import 'swiper/css/navigation'

// import required modules
import { Autoplay, Pagination, Navigation } from 'swiper/modules'

export default {
  components: {
    Swiper,
    SwiperSlide
  },
  setup() {
    return {
      modules: [Autoplay, Pagination, Navigation]
    }
  }
}
</script>

<template>
  <HeaderSection />
  <nav class="nav">
    <ul>
      <li><a href="#">홈</a></li>
      <li><a href="#">홈</a></li>
      <li><a href="#">홈</a></li>
      <li><a href="#">홈</a></li>
    </ul>
  </nav>

  <Modal ref="modalRef">
    <div v-if="movieVideo">
      <iframe
        width="560"
        height="315"
        :src="'https://www.youtube.com/embed/' + movieVideo.key + '?autoplay=1'"
        frameborder="0"
        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
      ></iframe>
    </div>
    <div v-else>비디오를 찾을 수 없습니다.</div>
  </Modal>

  <main id="main" role="main">
    <div class="view__inner container">
      <section class="view__card style1">
        <h3>최신 영화</h3>
        <div class="swiper-container card">
          <swiper
            :slidesPerView="5"
            :spaceBetween="20"
            :centeredSlides="false"
            :autoplay="{
              delay: 2500,
              disableOnInteraction: false
            }"
            :navigation="true"
            :modules="[Autoplay, Navigation]"
            class="mySwiper"
            :breakpoints="{
              320: { slidesPerView: 2 },
              768: { slidesPerView: 3 },
              1024: { slidesPerView: 4 },
              1200: { slidesPerView: 5 }
            }"
          >
            <swiper-slide
              class="slider"
              v-for="movie in latestMovies"
              :key="movie.id"
              @click="() => fetchVideo(movie.id)"
            >
              <img
                :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path"
                :alt="movie.title"
              />
              <span>{{ movie.title }}</span>
            </swiper-slide>
          </swiper>
        </div>
      </section>
      <section class="view__card style1">
        <h3>인기 영화</h3>
        <div class="card">
          <ul>
            <li
              class="card"
              v-for="movie in popularMovies"
              :key="movie.id"
              @click="() => fetchVideo(movie.id)"
            >
              <img
                :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path"
                :alt="movie.title"
              />
              <span>{{ movie.title }}</span>
              <p>인기 수 : {{ movie.popularity }}</p>
              <span>평점 : {{ movie.vote_average }}</span>
            </li>
          </ul>
        </div>
      </section>
    </div>
  </main>

  <FooterSection />
</template>

<style scoped>
* {
  color: var(--white);
}

.nav {
  ul {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}

#main {
  padding: 0 2rem 2rem 2rem;
}

.view__card {
  margin-top: 50px;

  h3 {
    font-family: 'theJamsil';
    font-weight: 300;
    margin-bottom: 10px;
    font-size: 26px;
    color: var(--white);
  }

  .swiper-container {
    width: 100%;
    height: 100%;
  }

  .card {
    ul {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 1rem;
    }

    .slider {
      border: 1px solid var(--black100);
      border-radius: 10px;
      overflow: hidden;
      background-color: var(--black200);
      width: 200px;
      height: 100%;
      box-sizing: border-box;

      img {
        width: 100%;
        height: 400px;
        object-fit: cover;
        cursor: pointer;
      }

      span,
      p {
        padding: 1rem;
        display: block;
        color: var (--white);
      }
    }
  }
}
</style>
