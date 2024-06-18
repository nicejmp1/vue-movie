<script setup>
import { defineProps, onMounted, ref } from 'vue'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'

const props = defineProps({
  movieID: {
    type: String,
    required: true
  }
})

// 영화 데이터 상태 관리
const movie = ref(null)
const formattedOverview = ref([])
const apikey = import.meta.env.VITE_TMDB_API_KEY

// 영화 정보 가져오기 함수
const fetchMovieDetails = async () => {
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/movie/${props.movieID}?api_key=${apikey}&language=ko`
    )
    movie.value = response.data
    // overview 텍스트 포맷팅
    formatOverview(movie.value.overview)
  } catch (error) {
    console.error('Error fetching movie details:', error)
  }
}

// overview 텍스트에 단어별로 줄바꿈 삽입 함수
const formatOverview = (text) => {
  if (text) {
    // 텍스트를 단어 단위로 분할
    const words = text.split(' ')
    const lines = []
    let line = []
    const maxWordsPerLine = 6 // 줄 당 최대 단어 수

    // 단어들을 순회하며 줄 단위로 묶기
    words.forEach((word, index) => {
      line.push(word)
      if ((index + 1) % maxWordsPerLine === 0) {
        lines.push(line.join(' '))
        line = []
      }
    })
    if (line.length > 0) {
      lines.push(line.join(' '))
    }

    formattedOverview.value = lines
  } else {
    formattedOverview.value = []
  }
}

// 컴포넌트 마운트 시 영화 정보 가져오기
onMounted(fetchMovieDetails)
</script>

<template>
  <HeaderSection />
  <div class="video__info" v-if="movie">
    <div class="video__main">
      <div class="video__post">
        <img :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path" alt="Movie Poster" />
      </div>
      <div class="video__title">
        <h1>{{ movie.title }}</h1>
      </div>
    </div>
    <div class="video__title__detail">
      <p v-for="(line, index) in formattedOverview" :key="index">{{ line }}</p>
      <span
        ><em>{{ movie.tagline }}</em></span
      >
    </div>
  </div>
  <div v-else>
    <p>로딩 중 ...</p>
  </div>
  <FooterSection />
</template>

<style lang="scss" scoped>
.video__info {
  padding: 2rem;

  .video__main {
    display: flex;
    margin-bottom: 2rem;

    .video__post {
      width: 200px;
      height: 300px;

      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
    }
    .video__title {
      margin-left: 1rem; /* 간격 추가 */

      h1 {
        font-size: 2rem; /* 예시 폰트 크기 */
        margin-bottom: 1rem; /* 예시 여백 */
      }
    }
  }
  .video__title__detail {
    p {
      white-space: normal;
      word-wrap: break-word;
      line-height: 2;
    }
    span {
      display: block;
      margin-top: 20px;

      em {
        font-size: 24px;
        font-weight: bold;
      }
    }
  }
}
</style>
