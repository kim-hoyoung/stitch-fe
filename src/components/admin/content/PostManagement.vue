<!--
 담당자: 김호영
 시작 일자: 2024.09.10
 설명 : admin 게시글 관리 페이지 기능 구현 및 디자인 개발
 ---------------------
 2024.09.10 김호영 | admin 초기 설정
 2024.09.13 김호영 | 초기 게시글 관리 디자인 및 기능 구현
 2024.09.19 김호영 | 날짜 출력형식 수정
 2024.09.25 김호영 | 게시글 백엔드 작업 및 게시판 버튼 추가 후 검색 카테고리 기능 수정.
 2024.10.01 김호영 | 게시판 필터 및 검색, 대분류 카테고리 기능 수정 + 삭제기능 구현.
 2024.10.04 김호영 | formData 형식 수정.
 -->

 <template>
  <div class="post-info-page">
    <!-- 대분류 게시판 카테고리 버튼 -->
    <div class="board-buttons" style="display: flex; justify-content: flex-start; margin-bottom: 20px;">
      <button @click="boardCategory = 'infoShareBoard'; loadInfoShareBoard()" class="board-button" :class="{ active: boardCategory === 'infoShareBoard' }">정보 공유</button>
      <button @click="boardCategory = 'free-community'; loadFreeCommunity()" class="board-button" :class="{ active: boardCategory === 'free-community' }">자유게시판</button>
      <button @click="boardCategory = 'qnABoard'; loadQnABoard()" class="board-button" :class="{ active: boardCategory === 'qnABoard' }">Q&A</button>
    </div>
    <!-- 상단 검색창 -->
    <div class="search-bar">
      <div class="search-wrapper search-container">

      <!-- 카테고리 선택 -->
      <div class="category-container">
        <select v-model="searchCategory" class="search-category"
          @focus="isDropdownOpen = true"
          @blur="isDropdownOpen = false"
          @change="isDropdownOpen = false">
          <option value="all">전체</option>
          <option value="nickname">작성자</option>
          <option value="type">게시글 유형</option>
          <option value="title">제목</option> 
        </select>
        <font-awesome-icon 
          :icon="isDropdownOpen ? ['fas', 'angle-up'] : ['fas', 'angle-down']" 
          class="angle-dropdown-icon" 
        />
      </div>
          
        <!-- 검색 icon + 검색창 -->
         <div class="search-input-container">
          <font-awesome-icon :icon="['fas', 'magnifying-glass']" class="search-icon" />
          <input type="text" v-model="searchQuery" placeholder="Search" class="search-input" />
        </div>
      </div>
    </div>

    <!-- 게시글 목록 테이블 -->
    <table class="post-list-table">
      <thead>
        <tr>
          <th>No.</th>
          <th>게시글 번호</th>
          <th>게시글 유형</th>
          <th>작성자</th>
          <th>등록일자</th>
          <th>수정 일자</th>
          <th>게시글 제목</th>
          <th>사용 여부</th>
          <!--<th>처리 상태</th>-->
          <th></th>
        </tr>
      </thead>
      <tbody>
          <!-- 게시글이 없을 때 문구 표시 -->
          <tr v-if="paginatedPosts.length === 0">
            <td colspan="8" style="text-align: center;">
              {{ emptyMessage }}
            </td>
          </tr>
        <tr v-for="(post, index) in paginatedPosts" :key="post.boardId">
          <td>{{  (currentPage -1) * postsPerPage + index + 1 }}</td>
          <td>{{ post.boardId || '-' }}</td>
          <td>{{ post.type || '-' }}</td>
          <td>{{ post.nickname || '-' }}</td>  
          <td>{{ formatDate(post.regdate) || '-' }}</td>
          <td>{{ formatDate(post.editdate) || '-' }}</td>
          <td>{{ post.title || '-' }}</td>
          <td>{{ post.use_yn || '-' }}</td>
          <td>
            <div class="dropdown-container" @click.stop="toggleDropdown(index)">
              <font-awesome-icon :icon="['fas', 'bars']" class="icon-bars" />
              <div v-if="openDropdownIndex === index" class="dropdown-menu">
                <ul>
                  <li @click="handleItemClick(post, 'public')">
                    <font-awesome-icon :icon="['fas', 'unlock']" class="modal-icon" /> 사용
                  </li>
                  <li @click="handleItemClick(post, 'private')">
                    <font-awesome-icon :icon="['fas', 'lock']" class="modal-icon" /> 미사용
                  </li>
                  <!--li @click="handleItemClick(post, 'item3')">
                    <font-awesome-icon :icon="['fas', 'question']" class="modal-icon" /> 항목
                  </li-->
                </ul>
              </div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 삭제 완료 모달 -->
    <div v-if="isHideModalOpen" class="modal-overlay">
      <div class="modal-content">
        <h3>게시글 상태가 변경되었습니다.</h3>
        <div class="modal-buttons">
          <button @click="isHideModalOpen = false">확인</button>
        </div>
      </div>
    </div>

  <!-- 성공 완료 모달 -->
  <div v-if="isHideSuccessModalOpen" class="modal-success-overlay">
    <div class="modal-success-content">
      <div class="modal-icon-container">
        <font-awesome-icon :icon="['fas', 'circle-check']" class="modal-success-icon" />
      </div>
      <p>상태 변경이 완료되었습니다</p>
    </div>
  </div>



    <!-- 페이지네이션 -->
    <div class="pagination">
      <button @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">이전</button>
      <span v-for="page in totalPages" :key="page">
        <button @click="goToPage(page)" :class="{ active: currentPage === page }">{{ page }}</button>
      </span>
      <button @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">다음</button>
    </div>
  </div>
</template>

<script>
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import axios from 'axios';

export default {
  components: {
    FontAwesomeIcon, // Font Awesome 컴포넌트 등록
  },
  data() {
    return {
      searchQuery: '',
      boardCategory: 'all',  // 게시판 대분류 선택에 사용할 변수
      selectedCategory: 'allBoard', // 게시판 대분류 선택에 사용할 변수 
      currentPage: 1,
      postsPerPage: 12,
      isDropdownOpen: false,
      openDropdownIndex: null,
      isDeleteModalOpen: false,
      isDeleteSuccessModalOpen: false,
      postToDelete: null,
      // 세 개의 게시판 데이터를 위한 배열
      communityData: [], 
      infoData: [],
      inquiryData: [],
    };
  },

  computed: {
    emptyMessage() {
      if (this.boardCategory === 'infoShareBoard') {
        return '정보공유 게시글이 없습니다.';
      } else if (this.boardCategory === 'free-community') {
        return '자유게시글이 없습니다.';
      } else if (this.boardCategory === 'qnABoard') {
        return 'Q&A 게시글이 없습니다.';
      } else {
        return '게시글이 없습니다.';
      }
    },
    // 모든 게시글을 하나의 배열로 병합
    allPosts() {
      return [
        ...this.communityData.map(post => ({ ...post, type: 'community' })),
        ...this.infoData.map(post => ({ ...post, type: 'info' })),
        ...this.inquiryData.map(post => ({ ...post, type: 'inquiry' })),
      ];
    },
    // 선택된 카테고리에 따라 게시글 필터링
    filteredPosts() {
      return this.allPosts.filter(post => {
        // 게시판 카테고리에 따른 필터링
        if (this.boardCategory === 'infoShareBoard' && post.type !== 'info') {
          return false;
        } else if (this.boardCategory === 'free-community' && post.type !== 'community') {
          return false;
        } else if (this.boardCategory === 'qnABoard' && post.type !== 'inquiry') {
          return false;
        }

        // 검색 카테고리와 검색어에 따른 필터링
        if (this.searchCategory === 'nickname') {
          return (post.nickname || '-').includes(this.searchQuery);
        } else if (this.searchCategory === 'type') {
          return (post.type || '-').includes(this.searchQuery);
        } else if (this.searchCategory === 'title') {
          return (post.title || '-').includes(this.searchQuery);
        } else {
          return (
            (post.title || '-').includes(this.searchQuery) ||
            (post.nickname || '-').includes(this.searchQuery) ||
            (post.content || '-').includes(this.searchQuery)
          );
        }
      });
    },
    // 현재 페이지에 표시할 게시글
    paginatedPosts() {
      const start = (this.currentPage - 1) * this.postsPerPage;
      const end = start + this.postsPerPage;
      return this.filteredPosts.slice(start, end);
    },
    // 전체 페이지 수
    totalPages() {
      return Math.ceil(this.filteredPosts.length / this.postsPerPage);
    },
  },
  mounted() {
    this.loadAllBoards();
  },
  methods: {
    formatDate(date) {
      if (!date) {
        return '-'; // 날짜가 null 또는 undefined이면 '-' 반환
      }
      const d = new Date(date);
      return d.toISOString().replace('T', ' ').substring(0, 10);
    },
    // 페이지 이동 메서드
    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
    // 드롭다운 토글
    toggleDropdown(index) {
      console.log('Current openDropdownIndex: ', this.openDropdownIndex);
      this.openDropdownIndex = this.openDropdownIndex === index ? null : index;
      console.log('Updated openDropdownIndex: ', this.openDropdownIndex);
    },
    // 게시글 숨김 확인 모달 열기 및 상태 변경 처리
    handleItemClick(post, action) {
      if (action === 'use') {
        this.changePostStatus(post, 'Y');  // 게시글 사용
      } else if (action === 'disuse') {
        this.changePostStatus(post, 'N');  // 게시글 미사용
      }
    },
    confirmDeletePost(post) {
      this.postToDelete = post;
      this.isDeleteModalOpen = true;
    },
   // 게시글 상태 변경 함수 
    async changePostStatus(post, status) {
      try {
        const token = localStorage.getItem('jwt');
        if (!token) {
          console.error('JWT 토큰이 없습니다.');
          return;
        }

        await axios.put(`/api/admin/board/status/${post.boardId}`, {
          useYn: status // 여기서 status는 'Y' 또는 'N'
        }, { 
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        post.useYn = status;

        this.isHideModalOpen = false;
        this.isHideSuccessModalOpen = true;

        setTimeout(() => {
          this.isHideSuccessModalOpen = false;
          this.postToHide = null;
        }, 1500);
      } catch (error) {
        console.error("게시글 비공개 처리 중 오류 발생: ", error);
      }
    },
    // 정보 공유 게시판 정보 불러오기
    loadInfoShareBoard() {
      this.selectedCategory = 'infoShareBoard';
      this.searchQuery = ''; // 검색어 초기화
      this.searchCategory = 'all'; // 검색 카테고리 초기화
      // 여기서 정보 공유 게시판 데이터를 불러오는 API 호출을 추가하면 됨
      console.log("정보 공유 게시판 데이터를 불러옵니다.");
    },
    // 자유게시판 정보 불러오기
    loadFreeCommunity() {
    this.selectedCategory = 'free-community';
    this.searchQuery = '';
    this.searchCategory = 'all';

    // 자유게시판 데이터를 불러오는 API 호출
    fetch('/api/board/community/all')
      .then(response => {
        if (!response.ok) {
          throw new Error('Failed to fetch posts');
        }
        return response.json();
      })
      .then(data => {
        this.communityData = data;
        console.log('자유 게시판 데이터를 성공적으로 불러왔습니다:', data);
      })
      .catch(error => {
        console.error('자유 게시판 데이터를 불러오는 데 실패했습니다:', error);
      });
      // 자유게시판 데이터를 불러오는 API 호출 추가
      console.log("자유 게시판 데이터를 불러옵니다.");
    },
    // Q&A 게시판 정보 불러오기
    loadQnABoard() {
      this.selectedCategory = 'qnABoard';
      this.searchQuery = '';
      this.searchCategory = 'all';
      // Q&A 게시판 데이터를 불러오는 API 호출 추가
      console.log("Q&A 게시판 데이터를 불러옵니다.");
    },

    // 모든 게시판 정보 불러오기
    loadAllBoards() {
      // 정보 공유 게시판 불러오기
      fetch('/api/board/infoShare/all')
        .then(response => {
          if (!response.ok) {
            throw new Error('Failed to fetch info share posts');
          }
          return response.json();
        })
        .then(data => {
          this.infoData = data;
          console.log('정보 공유 게시판 데이터를 성공적으로 불러왔습니다:', data);
        })
        .catch(error => {
          console.error('정보 공유 게시판 데이터를 불러오는 데 실패했습니다:', error);
        });

      // 자유 게시판 불러오기
      fetch('/api/board/community/all')
        .then(response => {
          if (!response.ok) {
            throw new Error('Failed to fetch community posts');
          }
          return response.json();
        })
        .then(data => {
          this.communityData = data;
          console.log('자유 게시판 데이터를 성공적으로 불러왔습니다:', data);
        })
        .catch(error => {
          console.error('자유 게시판 데이터를 불러오는 데 실패했습니다:', error);
        });

      // Q&A 게시판 불러오기
      fetch('/api/board/qna/all')
        .then(response => {
          if (!response.ok) {
            throw new Error('Failed to fetch Q&A posts');
          }
          return response.json();
        })
        .then(data => {
          this.inquiryData = data;
          console.log('Q&A 게시판 데이터를 성공적으로 불러왔습니다:', data);
        })
        .catch(error => {
          console.error('Q&A 게시판 데이터를 불러오는 데 실패했습니다:', error);
        });
    },
  },

};
</script>

<style scoped>


/* 게시판 카테고리 */
.board-button {
  background: none;
  border: none;
  color: #333;
  font-size: 12px;
  margin-right: 15px;
  cursor: pointer;
  padding: 5px;
  transition: color 0.3s;
}

.board-button:hover {
  color: #ff7f00; /* 호버 시 주황색 */
}

.board-button.active {
  color: #ff7f00; /* 활성화된 버튼 (클릭된 상태) 주황색 */
}

.board-button:focus {
  outline: none;
}


/* 드롭다운 스타일 */
.dropdown-container {
  position: relative;
  display: inline-block;
}

.icon-bars {
  cursor: pointer;
  color: #ababab;
}

/* 드롭다운 메뉴 */

.dropdown-menu {
  position: absolute;
  top: 100%;
  right: 0;
  background-color: white;
  border: 1.2px solid #ddd;
  border-radius: 5px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  min-width: 100px; /* 드롭다운의 최소 넓이 설정 */
  max-width: 140px; /* 드롭다운의 최대 넓이 설정 */
  width: auto; /* 내용에 따라 자동으로 조절 */
  margin-right: 10px;
}

.dropdown-menu ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.dropdown-menu li {
  display: flex;
  align-items: center;
  padding: 11px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.dropdown-menu li:hover {
  background-color: #efefef;
}

.modal-icon {
  color: #7b7b7b;
  margin-right: 10px;
  flex-shrink: 0; /* 아이콘이 항상 같은 크기로 고정되도록 */
}

/*state 모달 */

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.6); /* 배경을 어둡게 */
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1001; /* 위에 배치 */
}

/* 모달 콘텐츠 스타일 */
.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 300px;
  text-align: center;
}
.modal-content p {
  font-size: 14px;
  color: #7b7b7b;
  margin-bottom: 40px;
}

.modal-buttons {
  display: flex;
  justify-content: space-around;
  margin-top: 15px;
}

.modal-buttons button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
  width: 120px;
}

.modal-buttons button:first-child {
  background-color: #f56565;
  color: white;
}

.modal-buttons button:first-child:hover {
  background-color: #ec2727;
  color:white;
}

.modal-buttons button:last-child:hover {
  background-color: #b3b3b3; /* 호버 시 더 짙은 회색 */
}


/* 성공 모달 */
.modal-success-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1001; /* 위에 배치 */
}

.modal-success-content {
  display: flex;
    align-items: center;
    background-color: white;
    padding: 15px 25px;
    border-radius: 7px;
    box-shadow: 0px 1px 9px 3px rgba(0, 0, 0, 0.09);
    gap: 45px;
    width: 300px;
}

.modal-success-icon {
  color: #4F8CFF; /* 체크 아이콘 색상 */
  font-size: 30px;
  margin-right: 20px;
}

.modal-success-content p {
  margin: 0;
  font-size: 16px;
  color: #333;
}


/* 전체 레이아웃 스타일 */
.post-info-page {
  padding: 5px;
  position: relative; /* 페이지네이션을 하단에 고정시키기 위한 설정 */
  min-height: 100%; /* 컨테이너가 content-area와 동일한 높이를 가지도록 설정 */
  box-sizing: border-box; /* padding과 border를 포함한 높이 계산 */
}

/* content-area와 동일한 높이와 레이아웃으로 설정 */
.content-area {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  min-height: 100%; /* content-area의 높이 100% */
  box-sizing: border-box; /* padding과 border를 포함한 높이 계산 */
}

/* 페이지네이션 스타일 */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute; /* 부모 요소의 하단에 고정 */
  bottom: 10px; /* 부모 요소의 아래쪽에서 10px 떨어짐 */
  left: 0;
  right: 0;
  /*padding-bottom: 60px; /* 아래쪽에 추가적인 여백 */
}

.pagination button {
  padding: 10px 15px;
  margin: 0 5px;
  border: none;
  background-color: #f4f4f4;
  cursor: pointer;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.pagination button.active {
  background-color: #ffa15e;
  color: white;
}

.pagination button:disabled {
  background-color: #ddd;
}

.pagination button:hover:not(.active):not(:disabled) {
  background-color: #ffe5d1;
}

/* 창 스타일------------------------------------------------------------------ */
.category-container {
  position: relative;
  display: inline-flex;
  align-items: center;
}

.search-category {
  appearance: none; /* 기본 브라우저 스타일 제거 */
  margin-right: 20px;
  background-color: #ffffff; /* 배경색 */
  border: 1px solid #ccc; /* 경계선 */
  border-radius: 5px; /* 둥근 모서리 */
  padding: 8px 30px 8px 10px; /* 좌우 패딩 수정 (왼쪽 10px, 오른쪽 20px) */
  font-size: 12px; /* 폰트 크기 */
  font-weight: 500;
  color: #7b7b7b; /* 글자 색상 */
  cursor: pointer; /* 마우스 포인터 변경 */
  outline: none; /* 포커스 아웃라인 제거 */
  height: 100%; /* 높이 조절 */
  text-align: left; /* 텍스트 정렬을 왼쪽으로 */
}

.angle-dropdown-icon {
  position: absolute;
  right: 30px; /* 아이콘을 오른쪽에 배치 */
  font-size: 14px;
  color: #7b7b7b;
}


.search-bar {
  margin-bottom: 20px;
}

.search-wrapper {
  display: flex; /* 검색창이랑 카테고리 가로로 배치 */
  align-items: center;
  width: 60%; /* 검색창 + 카테고리 창의 너비 */
}


.search-wrapper input {
  width: 100%;
  padding: 10px 10px 10px 35px; /* 아이콘과 텍스트 간 여백을 위한 padding */
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: border-color 0.3s;
}

/* 검색창에 포커스가 들어올 때 */
.search-input:focus {
  outline: none; /* 기본 아웃라인 제거 */
  border-color: #F8A060; /* 포커스 시 테두리 색상 변경 */
}

.search-input:focus + .clear-btn {
  /* clear-btn이 있을 경우 추가적인 스타일을 변경 */
  display: block; /* clear 버튼을 보이게 만듦 */
}

.search-container:focus-within {
  border-color: #F8A060; /* 검색창에 포커스가 있을 때 컨테이너 테두리 색상 변경 */
}

/* 검색 아이콘 스타일 */
.search-input-container {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
}

.search-icon {
  position: absolute;
  left: 10px; /* 아이콘의 왼쪽 위치 */
  top: 50%;
  transform: translateY(-50%); /* 아이콘을 수직 중앙으로 */
  font-size: 16px; /* 아이콘 크기 설정 */
  color: #8f8f8f; /* 아이콘 색상 */
}



/* 게시글 목록 테이블 스타일 ---------------------------------------------------------*/
.post-list-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 50px;
}

/* 첫 번째와 두 번째 열(순번과 게시글 번호) 패딩 조정 */
.post-list-table th:nth-child(1), .post-list-table th:nth-child(2), 
.post-list-table td:nth-child(1), .post-list-table td:nth-child(2) {
  padding-left: 5px; /* 좌측 패딩 값 줄이기 */
  padding-right: 5px; /* 우측 패딩 값 줄이기 */
}

/* 테이블 헤더 스타일 */
.post-list-table th {
  font-weight: bold;
  font-size: 12px; 
  padding: 0px 10px 15px 20px; /* 상, 좌, 우, 하 padding 설정 */
  text-align: center;
  border-bottom: 1px solid #575757;
}

/* 테이블 데이터 스타일 */
.post-list-table td {
  font-size: 12px; /* td에 원하는 크기 설정 */
  padding: 18px 10px 15px 20px; /* 상, 좌, 우, 하 padding 설정 */
  text-align: center;
}

.icon-bars {
  color: #ababab; /* 아이콘 색상 변경 */
  font-size: 15px;
  font-weight:300;
}



/* 반응형 디자인 */
@media (max-width: 1024px) {
  /* 테이블 헤더 스타일 */
  .post-list-table th {
    font-size: 10px; 
    padding: 0px 10px 12px 15px; /* 상, 좌, 우, 하 padding 설정 */
  }
  /* 테이블 데이터 스타일 */
  .post-list-table td {
    font-size: 10px; /* td에 원하는 크기 설정 */
  }
}


@media (max-width: 768px) {
  /* 테이블 헤더 스타일 */
  .post-list-table th {
    font-size: 8px; 
    padding: 0px 8px 12px 15px; /* 상, 좌, 우, 하 padding 설정 */
  }
  /* 테이블 데이터 스타일 */
  .post-list-table td {
    font-size: 8px; /* td에 원하는 크기 설정 */
  }
}

@media (max-width: 480px) {
  /* 테이블 헤더 스타일 */
  .post-list-table th {
    font-size: 6px; 
    padding: 0px 6px 10px 12px; /* 상, 좌, 우, 하 padding 설정 */
  }
  /* 테이블 데이터 스타일 */
  .post-list-table td {
    font-size: 6px; /* td에 원하는 크기 설정 */
  }
  
}
</style>