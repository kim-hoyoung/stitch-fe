<!-- 
 담당자: 박요한
 시작 일자: 2024.09.13
 설명 : 조회수 높은 게시글 목록
 ---------------------
 2024.09.13 박요한 | 컴포넌트 생성.
 2024.09.18 박요한 | 구체화.
 2024.09.25 박요한 | 더보기 버튼 위치 조정.
 2024.09.26 박요한 | 데이터 인덱싱 수정. 카드 높이 조정. 라우터 연결.
 -->

<template>
  <div class="popular-posts">
    <h2>인기 게시글</h2>
    <div class="board-container">
      <!-- 정보 공유 게시판 -->
      <div class="board-section" v-if="infoSharePost">
        <div class="section-header">
          <h3>정보 공유</h3>
          <more-button to="/board/info-share" />
        </div>
        <div class="post-card" @click="$router.push(`/board/info-share/post/${infoSharePost.boardId}`)">
          <p class="post-title">{{ infoSharePost.title }}</p>
          <p class="post-views">조회수: {{ infoSharePost.views }}</p>
        </div>
      </div>

      <!-- 자유 게시판 -->
      <div class="board-section" v-if="freeCommunityPost">
        <div class="section-header">
          <h3>자유 게시판</h3>
          <more-button to="/board/free-community" />
        </div>
        <div class="post-card" @click="$router.push(`/board/free-community/post/${freeCommunityPost.boardId}`)">
          <p class="post-title">{{ freeCommunityPost.title }}</p>
          <p class="post-views">조회수: {{ freeCommunityPost.views }}</p>
        </div>
      </div>

      <!-- Q&A 게시판 -->
      <div class="board-section" v-if="qnaPost">
        <div class="section-header">
          <h3>Q&A 게시판</h3>
          <more-button to="/board/qna" />
        </div>
        <div class="post-card" @click="$router.push(`/board/qna/post/${qnaPost.boardId}`)">
          <p class="post-title">{{ qnaPost.title }}</p>
          <p class="post-views">조회수: {{ qnaPost.views }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import MoreButton from "./MoreButton.vue";

export default {
  components: { MoreButton },
  name: "PopularPosts",
  data() {
    return {
      infoSharePost: { title: "정보 공유 인기 게시글", views: 10 },
      freeCommunityPost: { title: "자유 게시판 인기 게시글", views: 10 },
      qnaPost: { title: "Q&A 인기 게시글", views: 10 },
    };
  },
  mounted() {
    this.fetchPopularPosts();
  },
  methods: {
    async fetchPopularPosts() {
      try {
        // 정보 공유 게시판 인기 게시글 가져오기
        const infoShareResponse = await axios.get("/api/home/posts/info-share/top");
        this.infoSharePost = infoShareResponse.data[0];
      } catch (error) {
        console.error("Error fetching info share post:", error);
      }

      try {
        // 자유 게시판 인기 게시글 가져오기
        const freeCommunityResponse = await axios.get("/api/home/posts/free-community/top");
        this.freeCommunityPost = freeCommunityResponse.data[0];
      } catch (error) {
        console.error("Error fetching free community post:", error);
      }

      try {
        // Q&A 게시판 인기 게시글 가져오기
        const qnaResponse = await axios.get("/api/home/posts/qna/top");
        this.qnaPost = qnaResponse.data[0];
      } catch (error) {
        console.error("Error fetching Q&A post:", error);
      }
    },
  },
};
</script>

<style scoped>
.popular-posts {
  margin: 0 auto;
}

h2 {
  padding: 1%;
}

.board-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.board-section {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-bottom: 20px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  padding: 1%;
  margin-left: 3%;
  margin-right: 3%;
}

.section-header h3 {
  margin: 0;
}

.post-card {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border: 1px solid #ddd;
  padding: 25px;
  margin-bottom: 20px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.3s ease;
}

.post-card .post-title {
  flex-grow: 1; /* 제목 부분이 남은 공간을 차지하게 함 */
  margin-bottom: 20px;
}

.post-card .post-views {
  margin-top: auto; /* 조회수를 카드 하단에 위치시키기 */
  text-align: right;
  font-size: 14px;
  color: #888;
}

.post-card:hover {
  transform: scale(1.05);
}
</style>
