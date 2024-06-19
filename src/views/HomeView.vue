<script setup>
import { onMounted, ref, watch, onBeforeUnmount } from 'vue'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'
import Modal from '@/components/ModalSection.vue'
import SvgIcon from '@jamescoyle/vue-icon'
import { mdiVolumeHigh, mdiVolumeOff, mdiPlay, mdiPause, mdiStar } from '@mdi/js'

const pathVolumeHigh = ref(mdiVolumeHigh)
const pathVolumeOff = ref(mdiVolumeOff)
const pathPlay = ref(mdiPlay)
const pathPause = ref(mdiPause)
const pathStart = ref(mdiStar)

const latestMovies = ref([])
const popularMovies = ref([])
const movieVideo = ref(null)
const modalRef = ref(null)
const movieVideos = ref([])
const apikey = import.meta.env.VITE_TMDB_API_KEY

const movieIds = [1022789, 748783, 653346] // 원하는 영화 ID 리스트

let currentVideoIndex = ref(0)
let currentVideo = ref(JSON.parse(localStorage.getItem('currentVideo')) || null)
let player = null
let isMuted = ref(true)
let isPlaying = ref(false)

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
      `https://api.themoviedb.org/3/movie/popular?api_key=${apikey}&language=ko&page=1`
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

const fetchAllVideos = async () => {
  try {
    const promises = movieIds.map((id) =>
      axios.get(`https://api.themoviedb.org/3/movie/${id}/videos?api_key=${apikey}&language=ko`)
    )
    const responses = await Promise.all(promises)
    movieVideos.value = responses.map((response, index) => {
      const videos = response.data.results.filter((video) => video.site === 'YouTube')
      videos.forEach((video) => {
        const movie = latestMovies.value
          .concat(popularMovies.value)
          .find((movie) => movie.id === movieIds[index])
        if (movie) {
          video.title = movie.title
          video.overview = movie.overview
          video.vote_average = movie.vote_average
        }
      })
      return videos
    })
    if (movieVideos.value.flat().length > 0) {
      currentVideo.value = movieVideos.value.flat()[0]
      loadPlayer()
    }
    console.log(movieVideos.value)
  } catch (error) {
    console.error(error)
  }
}

const nextVideo = () => {
  if (movieVideos.value.flat().length > 0) {
    currentVideoIndex.value = (currentVideoIndex.value + 1) % movieVideos.value.flat().length
    currentVideo.value = movieVideos.value.flat()[currentVideoIndex.value]
    localStorage.setItem('currentVideo', JSON.stringify(currentVideo.value))
    if (player) {
      player.loadVideoById(currentVideo.value.key)
    }
  }
}

const loadPlayer = () => {
  if (currentVideo.value) {
    if (window.YT) {
      player = new window.YT.Player('video-player', {
        width: '1000px',
        height: '900px',
        videoId: currentVideo.value.key,
        playerVars: { controls: 0 }, // 사용자 컨트롤 숨기기
        events: {
          onReady: onPlayerReady,
          onStateChange: onPlayerStateChange
        }
      })
    } else {
      const tag = document.createElement('script')
      tag.src = 'https://www.youtube.com/iframe_api'
      const firstScriptTag = document.getElementsByTagName('script')[0]
      firstScriptTag.parentNode.insertBefore(tag, firstScriptTag)
      window.onYouTubeIframeAPIReady = () => {
        loadPlayer()
      }
    }
  }
}

const onPlayerReady = (event) => {
  event.target.playVideo()
  event.target.setVolume(0) // 볼륨 0으로 설정
  isPlaying.value = true
  isMuted.value = true
}

const onPlayerStateChange = (event) => {
  if (event.data === window.YT.PlayerState.ENDED) {
    nextVideo()
  }
}

const toggleVolume = () => {
  if (player) {
    const currentVolume = player.getVolume()
    if (currentVolume === 0) {
      player.setVolume(50) // 볼륨 키기
      isMuted.value = false
    } else {
      player.setVolume(0) // 볼륨 끄기
      isMuted.value = true
    }
  }
}

const togglePlayPause = () => {
  if (player) {
    if (isPlaying.value) {
      player.pauseVideo() // 비디오 일시정지
      isPlaying.value = false
    } else {
      player.playVideo() // 비디오 재생
      isPlaying.value = true
    }
  }
}

// 컴포넌트가 마운트될 때 비디오 플레이어를 다시 로드
const reloadVideoPlayer = () => {
  if (currentVideo.value) {
    loadPlayer()
  }
}

onMounted(fetchMovies)
onMounted(fetchPopular)
onMounted(fetchAllVideos)
onMounted(reloadVideoPlayer) // 비디오 플레이어 다시 로드

// watch를 사용하여 currentVideo가 변경될 때마다 loadPlayer를 호출
watch(currentVideo, (newVideo) => {
  if (newVideo) {
    loadPlayer()
    localStorage.setItem('currentVideo', JSON.stringify(newVideo))
  }
})

// 페이지가 언마운트될 때 비디오 플레이어를 초기화
onBeforeUnmount(() => {
  if (player) {
    player.destroy()
    player = null
  }
})
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
  <section>
    <article>
      <div class="main__video">
        <div v-if="currentVideo">
          <div id="video-player" class="video"></div>
          <div class="video-overlay">
            <div class="video__title">
              <h2>{{ currentVideo.title }}</h2>
              <span
                ><svg-icon type="mdi" :path="pathStart"></svg-icon
                >{{ currentVideo.vote_average }}</span
              >
            </div>
            <p>{{ currentVideo.overview }}</p>
            <button @click="toggleVolume" class="volume-button">
              <svg-icon type="mdi" :path="isMuted ? pathVolumeOff : pathVolumeHigh"></svg-icon>
            </button>
            <button @click="togglePlayPause" class="play-button">
              <svg-icon type="mdi" :path="isPlaying ? pathPause : pathPlay"></svg-icon>
            </button>
          </div>
        </div>
        <div v-else>비디오를 찾을 수 없습니다.</div>
      </div>
    </article>
  </section>

  <Modal ref="modalRef">
    <div v-if="movieVideo" class="video-container">
      <iframe
        width="100%"
        height="100%"
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
            <swiper-slide class="slider" v-for="movie in latestMovies" :key="movie.id">
              <img
                :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path"
                :alt="movie.title"
              />
              <span>{{ movie.title }}</span>
              <p>개봉일 : {{ movie.release_date }}</p>
              <p><svg-icon type="mdi" :path="pathStart"></svg-icon> {{ movie.vote_average }}</p>
              <div class="movieChart_btn_wrap">
                <router-link
                  :to="{ name: 'detail', params: { movieID: movie.id } }"
                  class="btn_movie_detail"
                  >상세보기</router-link
                >
                <a href="#" @click.prevent="fetchVideo(movie.id)" class="btn_movie_video"
                  >티저영상</a
                >
              </div>
            </swiper-slide>
          </swiper>
        </div>
      </section>
      <section class="view__card style1">
        <h3>인기 영화</h3>
        <div class="swiper-container card">
          <swiper
            :slidesPerView="3"
            :spaceBetween="10"
            :centeredSlides="false"
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
            <swiper-slide class="slider" v-for="movie in popularMovies" :key="movie.id">
              <img
                :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path"
                :alt="movie.title"
              />
              <span>{{ movie.title }}</span>
              <p>개봉일 : {{ movie.release_date }}</p>
              <p><svg-icon type="mdi" :path="pathStart"></svg-icon> {{ movie.vote_average }}</p>
              <div class="movieChart_btn_wrap">
                <router-link
                  :to="{ name: 'detail', params: { movieID: movie.id } }"
                  class="btn_movie_detail"
                  >상세보기</router-link
                >
                <a href="#" @click.prevent="fetchVideo(movie.id)" class="btn_movie_video"
                  >티저영상</a
                >
              </div>
            </swiper-slide>
          </swiper>
        </div>
      </section>
    </div>
  </main>

  <FooterSection />
</template>

<style lang="scss">
.main__video {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--black);
  position: relative;
  border-top: 4px solid #f00;

  &::before {
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
  }

  .video-overlay {
    position: absolute;
    left: 0;
    bottom: 20px;
    z-index: 10;
    background-color: rgba(0, 0, 0, 0.5);
    padding: 0 8rem 0 8rem;
    color: var(--white);
    border-radius: 5px;

    button {
      margin-right: 10px;
      background-color: var(--pointColor);
      color: var(--white);
      margin-top: 10px;
      padding: 4px;
      border-radius: 50%;
    }
    svg {
      cursor: pointer;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      line-height: 30px;
    }
  }

  .video__title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    h2 {
      font-size: 30px;
      font-family: var(--fontN);
      font-weight: 900;
    }
    span {
      font-size: 1.2rem;

      svg {
        vertical-align: -3px;
        width: 20px;
        height: 20px;
        margin-right: 6px;
        color: #f00;
      }
    }
  }

  .video-overlay p {
    margin: 5px 0;
    font-size: 1rem;
    font-family: var(--fontN);
    font-weight: 400;
    color: var(--white300);
  }
}

.video-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
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
    font-size: 30px;
    color: var(--black);
    margin-bottom: 20px;
  }

  .swiper-container {
    width: 100%;
    height: 100%;
  }

  .card {
    font-family: var(--fontN);
    ul {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 1rem;
    }

    .slider {
      border-radius: 10px;
      overflow: hidden;
      background-color: var(--black200);
      width: 200px;
      height: 100%;
      box-sizing: border-box;
      position: relative;

      img {
        width: 100%;
        height: 400px;
        object-fit: cover;
        cursor: pointer;
      }
      &::before {
        content: '';
        background-color: rgba(0, 0, 0, 0.4);
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        z-index: 100;
        display: none;
      }

      &:hover::before {
        display: block;
      }

      &:hover .movieChart_btn_wrap {
        display: flex;
      }

      span {
        text-align: center;
        padding: 1rem;
        display: block;
        color: var(--white);
        font-family: var(--fontN);
        font-weight: 500;
        font-size: 1rem;
      }
      p {
        text-align: center;
        padding: 0 1rem 1rem 1rem;
        display: block;
        color: var(--white);
        font-family: var(--fontN);
        font-size: 14px;

        svg {
          width: 16px;
          height: 16px;
          vertical-align: -2px;
          color: #f00;
        }
      }
    }
    .movieChart_btn_wrap {
      display: none;
      position: absolute;
      left: 50%;
      top: 50%;
      z-index: 100;
      transform: translate(-50%, -50%);
      flex-direction: column;
      align-items: center;

      .btn_movie_detail {
        color: var(--black);
        font-size: 20px;
        background-color: var(--white);
        padding: 4px 20px;
        border-radius: 10px;
        display: block;
        margin-bottom: 1rem;
        transition: all 0.3s;
        font-family: var(--fontN);

        &:hover {
          background-color: rgba(0, 0, 0, 0.8);
          color: var(--white);
        }
      }
      .btn_movie_video {
        background-color: var(--pointColor);
        color: var(--white);
        padding: 4px 20px;
        border-radius: 10px;
        font-size: 20px;
        display: block;
        transition: all 0.3s;

        &:hover {
          background-color: var(--oppColor);
        }
      }
    }
  }
}
</style>
