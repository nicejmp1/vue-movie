<template>
  <footer id="footer" role="contentinfo">
    <div class="footer__inner container">
      <span>해당 사이트는 교육목적으로 만든 사이트이며</span>
      <span>모드 영화 데이터는 TMDB 사이트에서 가져왔습니다.</span>
      <a href="mailto:nicejpm@naver.com">nicejpm@naver.com</a>
    </div>
  </footer>
</template>

<style lang="scss">
#footer {
  margin-top: 2rem;
  border-top: 1px solid var(--black);
  height: 100px;
  display: flex;
  align-items: center;
  background-color: var(--black400);

  .footer__inner {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    color: var(--white200);

    span {
      font-size: 1rem;
    }
  }
}
</style>
