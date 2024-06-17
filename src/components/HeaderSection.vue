<template>
  <header id="header" role="banner">
    <div class="header_inner container">
      <h1>MovieVerse</h1>
      <div class="search">
        <div class="search-box">
          <input type="search" placeholder="" />
          <button>검색</button>
        </div>
      </div>
    </div>
  </header>
  <!-- //header -->
</template>

<style lang="scss">
#header {
  background-color: #333;
}

.header_inner {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100px;
  color: #fff;
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
        color: #fff;
        cursor: pointer;
      }
    }
  }
  h1 {
    font-size: 2.5rem;
    span {
      font-size: 1.5rem;
    }
  }
}
</style>
