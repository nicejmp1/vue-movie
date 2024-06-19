<script setup>
import { computed, onMounted, ref } from 'vue'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'
import Modal from '@/components/ModalSection.vue'
import defaultImg from '@/assets/img/image.png' // 기본 이미지 가져오기

const props = defineProps({
  movieID: {
    type: String,
    required: true
  }
})

// 영화 데이터 상태 관리
const movie = ref(null)
const credits = ref(null)
const formattedOverview = ref([])
const movieVideoKey = ref(null)
const apikey = import.meta.env.VITE_TMDB_API_KEY

const filteredJobs = ['Director', 'Producer']

// 모달 참조
const modalRef = ref(null)

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
    const words = text.split(' ')
    const lines = []
    let line = []
    const maxWordsPerLine = 6

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

const fetchMovieCredits = async () => {
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/movie/${props.movieID}/credits?api_key=${apikey}&language=ko`
    )
    credits.value = response.data
    console.log(credits.value)
  } catch (error) {
    console.log(error)
  }
}

const fetchVideo = async (movie_id) => {
  try {
    const videoResponse = await axios.get(
      `https://api.themoviedb.org/3/movie/${movie_id}/videos?api_key=${apikey}&language=ko`
    )
    const videos = videoResponse.data.results
    if (videos.length > 0) {
      movieVideoKey.value = videos[0].key // 첫 번째 비디오의 키 저장
      modalRef.value.showModal() // 모달을 표시
    } else {
      movieVideoKey.value = null // 비디오가 없으면 null로 설정
    }
  } catch (error) {
    console.error(error)
    movieVideoKey.value = null // 오류 발생 시 null로 설정
  }
}

// 특정 job을 가진 크루 멤버를 job별로 그룹화하고 중복 제거
const groupedCrew = computed(() => {
  const groups = {
    Director: [],
    Acting: [],
    Producer: []
  }

  const uniqueCrew = new Set()
  const uniqueCast = new Set()

  if (credits.value && credits.value.crew) {
    credits.value.crew.forEach((member) => {
      if (filteredJobs.includes(member.job) && !uniqueCrew.has(member.id)) {
        groups[member.job].push(member)
        uniqueCrew.add(member.id)
      }
    })
  }

  if (credits.value && credits.value.cast) {
    credits.value.cast.forEach((member) => {
      if (!uniqueCast.has(member.id)) {
        groups.Acting.push(member)
        uniqueCast.add(member.id)
      }
    })
  }

  return groups
})

// 컴포넌트 마운트 시 영화 정보 가져오기
onMounted(fetchMovieDetails)
onMounted(fetchMovieCredits)
</script>

<template>
  <HeaderSection />
  <div class="video__info" v-if="movie">
    <div class="container">
      <div class="video__main">
        <div class="video__post">
          <img :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path" alt="Movie Poster" />
        </div>
        <div class="video__title">
          <h1>{{ movie.title }}</h1>
          <span>
            장르 :
            <span v-for="genre in movie.genres" :key="genre.id" class="genre">
              {{ genre.name }}
            </span>
          </span>
          <span v-if="groupedCrew.Director.length > 0"
            >감독 :
            <span v-for="member in groupedCrew.Director" :key="member.credit_id">
              {{ member.name }}
            </span>
          </span>
          <span v-if="groupedCrew.Producer.length > 0"
            >프로듀서 :
            <span v-for="member in groupedCrew.Producer" :key="member.credit_id">
              {{ member.name }}
            </span>
          </span>
          <span v-if="groupedCrew.Acting.length > 0"
            >배우 :
            <span
              v-for="member in groupedCrew.Acting.slice(0, 6)"
              :key="member.credit_id"
              class="member"
            >
              {{ member.name }}
            </span>
          </span>
          <span> 개봉 : {{ movie.release_date }} </span>
          <span>평점 : {{ movie.vote_average }}</span>
          <span>상영시간 : {{ movie.runtime }}분</span>
        </div>
      </div>
      <div class="video__title__detail">
        <p v-for="(line, index) in formattedOverview" :key="index">{{ line }}</p>
        <span>
          <em>{{ movie.tagline }}</em>
        </span>
        <button @click="fetchVideo(props.movieID)">티저영상</button>
      </div>
      <div class="video__detail">
        <h2>감독</h2>
        <div v-if="credits && credits.crew" class="crew">
          <div
            v-for="member in credits.crew.filter((m) => filteredJobs.includes(m.job))"
            :key="member.credit_id"
            class="crew-member"
          >
            <img
              :src="
                member.profile_path
                  ? 'https://image.tmdb.org/t/p/w500/' + member.profile_path
                  : defaultImg
              "
              alt="Crew Member Image"
            />
            <span>{{ member.name }}</span>
          </div>
        </div>
        <h2>배우</h2>
        <div v-if="credits && credits.cast" class="crew">
          <div
            v-for="member in credits.cast.slice(0, 6)"
            :key="member.credit_id"
            class="crew-member"
          >
            <img
              :src="
                member.profile_path
                  ? 'https://image.tmdb.org/t/p/w500/' + member.profile_path
                  : defaultImg
              "
              alt="Cast Member Image"
            />
            <span>{{ member.name }}</span>
          </div>
        </div>
        <div v-else>
          <p>크루 데이터를 불러오는 중...</p>
        </div>
      </div>
    </div>
  </div>
  <div v-else>
    <p>로딩 중 ...</p>
  </div>
  <Modal ref="modalRef">
    <div class="video-container" v-if="movieVideoKey">
      <iframe :src="'https://www.youtube.com/embed/' + movieVideoKey" allowfullscreen></iframe>
    </div>
  </Modal>
  <FooterSection />
</template>

<style lang="scss" scoped>
.member {
  margin-right: 10px;
}
.genre {
  margin-right: 10px;
}
.video__info {
  padding: 1rem 2rem 1rem 2rem;
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
        border-radius: 10px;
      }
    }
    .video__title {
      margin-left: 1rem; /* 간격 추가 */
      display: flex;
      flex-direction: column;
      padding-bottom: 20px;

      h1 {
        font-size: 2rem; /* 예시 폰트 크기 */
        margin-bottom: 1rem; /* 예시 여백 */
        font-family: var(--fontN);
        font-weight: 900;
      }
      span {
        padding-bottom: 10px;
      }
    }
  }
  .video__title__detail {
    button {
      font-size: 1.25rem;
      padding: 1px 10px;
      background-color: var(--pointColor);
      font-family: var(--fontN);
      color: var(--white);
      border-radius: 30px;
      margin-bottom: 30px;
      transition: all 0.3s;
      cursor: pointer;

      &:hover {
        background-color: var(--oppColor);
        color: var(--black);
      }
    }

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
      padding-bottom: 20px;
    }
    border-bottom: 1px solid var(--black);
  }
  .video__detail {
    margin-top: 30px;
    .crew {
      display: flex;
      flex-wrap: wrap;
    }
    .crew-member {
      display: flex;
      align-items: center;
      flex-direction: column;
      margin-bottom: 10px;
      margin-right: 10px;

      img {
        width: 100px;
        height: 150px;
        border-radius: 10px;
        object-fit: cover;
        margin-bottom: 5px;
      }
      span {
        font-size: 0.9rem;
        font-weight: 500;
        text-align: center;
      }
    }
  }
}
.video-container {
  width: 100%;
  height: 100%;
}

iframe {
  width: 100%;
  height: 100%;
  border: none;
}
</style>
