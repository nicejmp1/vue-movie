<template>
  <header id="header" role="banner">
    <div class="container">
      <div class="header_inner">
        <h1><router-link :to="{ name: 'home' }">Movie World</router-link></h1>
        <div class="search">
          <div class="search-box">
            <input
              type="search"
              v-model="query"
              placeholder="영화를 검색해보세요!!"
              @keyup.enter="search"
            />
            <button @click="search">검색</button>
          </div>
        </div>
      </div>
    </div>
  </header>
  <!-- //header -->
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const query = ref('')
const router = useRouter()

const search = () => {
  if (query.value.trim()) {
    router.push({ name: 'search', query: { q: query.value } })
  }
}
</script>

<style lang="scss">
#header {
  background-color: var(--mainBg);
  padding: 1rem 2rem 1rem 2rem;
}
.nav {
  ul {
    display: flex;
    font-size: 20px;
    font-family: var(--fontN);
    font-weight: 400;
    li {
      padding: 1px 40px 1px 1px;
    }
  }
}

.header_inner {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: var(--black);
  margin-bottom: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid var(--black300);

  .search {
    margin-left: auto;
    .search-box {
      display: flex;
      align-items: center;
      input {
        padding: 0.5rem;
        border: 1px solid #fff;
        border-radius: 5px 0 0 5px;
        border-right: none;
      }
      button {
        padding: 0.5rem 1rem;
        border: 1px solid #fff;
        border-radius: 0 5px 5px 0;
        background-color: transparent;
        color: var(--white);
        background-color: var(--pointColor);
        cursor: pointer;
      }
    }
  }
  h1 {
    font-size: 2.5rem;
    color: var(--pointColor);
    font-family: var(--fontN);
    font-weight: bold;
    span {
      font-size: 1.5rem;
    }
  }
}
</style>
