# 🖥️ Vue를 이용한 영화 소개 사이트
![image](https://github.com/nicejmp1/vue-movie/assets/163364733/40c69dff-4cfd-4a24-af22-939ccd6b52bf)

이 프로젝트는 Vue와 ,TMDB API 데이터를 활용하여 만든 사이트 입니다. 

# vue 설치 방법 
````
npm init vue@latest
Project name: .
yes -> vue -> no -> yes -> yes -> no -> no -> Y -> Y -> N

npm i axios 
npm i scss
npm i oh-vue-icons
````

# 사용한 기술 - api 
- TMDB Api (https://developer.themoviedb.org/docs/getting-started)
- Vue.js , Scss, HTML, Swiper

# 제작 기간 
- 2024.06.17 ~ 2024.06.19

# 주요 기능 
- **자동 슬라이드** : 최신 영화를 자동으로 넘기며 보여줍니다.
- **슬라이드 기능** : Swiper를 활용하여 영화 목록을 넘기는 기능을 구현하였습니다.
- **비디오 재생** : TMDB API를 활용하여 티저 영상을 소개합니다.
- **검색 기능** : 사용자가 원하는 영화 정보를 검색하여 찾을 수 있게 구현하였습니다.
- **상세 페이지** : 상세 페이지를 제작하여 보고 싶은 영화의 정보, 티저 영상, 배우 및 감독 정보를 제공합니다.

# 메인 페이지 디자인
![image](https://github.com/nicejmp1/vue-movie/assets/163364733/2febcf40-c5df-4f0a-9baf-fa9100979156)
> Swiper를 활용하여 자동 슬라이드를 구현하였고 슬라이드 기능을 통해 영화 목록을 넘기는 기능을 구현 하였습니다.

<br><br>

# 상세 페이지 디자인
![image](https://github.com/nicejmp1/vue-movie/assets/163364733/6137a5fa-b105-435c-8f93-2bb2934829dd)
> TMDB API 공식문서에 있는 영화 정보의 데이터들을 상세하게 정리 하였습니다.

<br><br>

# 비디오 재생
![image](https://github.com/nicejmp1/vue-movie/assets/163364733/2db290df-a79d-4a2e-bf28-a33ba45d122b)
> 사용자가 티저 영상을 클릭했을 때, 해당 movieID에 맞는 티저 영상이 모달로 표시되도록 구현하였습니다.

<br><br>

# 검색 기능
![image](https://github.com/nicejmp1/vue-movie/assets/163364733/2cb550d8-8aeb-4a62-8587-d0d7b718f4d7)
> 사용자가 원하는 영화를 검색하여 보기 편하게 만들었고, 해당 포스터를 누르면 상세 페이지로 이동하도록 구현하였습니다.

<br><br>




