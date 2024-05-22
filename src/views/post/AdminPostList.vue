<template>
  <main class="mt-5 mb-5">
    <div class="container mb-5">
      <div class="row d-flex justify-content-center">
          <h3 class="mt-3" style="font-weight: bold">정책정보관리</h3>
        <div class="col-12 d-flex justify-content-center mb-4">
          <!-- 카테고리 선택 드롭다운 -->
          <select class="form-select form-select mt-3" style="width:330px;" @change="handleCategoryChange($event)">
            <option value="전체">전체</option>
            <option value='보육'>보육</option>
            <option value="출산">출산</option>
            <option value="주거">주거</option>
            <option value="장례">장례</option>
          </select>
        </div>
        <div class="d-flex justify-content-end mt-2">
           <a href="/admin/post/register">
            <button class="btn btn-dark" style="color:white;"><b>정책정보 등록</b></button>
          </a>
        </div>
      </div>
      <!-- 게시물 테이블 -->
      <div class="table-responsive mt-3 d-flex justify-content-center">
        <table class="table" style="background-color: white;">
          <!-- 테이블 헤더 -->
          <thead>
            <tr>
              <th scope="col">선택</th> <!-- 체크박스를 추가한 열 -->
              <th scope="col">번호</th>
              <th scope="col">제목</th>
              <th scope="col">카테고리</th>
              <th scope="col">등록일자</th>
              <th scope="col">작성자</th>
              <th scope="col"></th>
            </tr>
          </thead>
          <!-- 게시물 목록 표시 -->
          <tbody style="background-color: white;">
            <tr v-for="(post, index) in postList" :key="index">
              <td>
                <!-- 체크박스 추가 -->
                <div style="padding: 10px 0;">
                <input type="checkbox" :id="'postCheckbox_' + index" v-model="selectedPosts" :value="post.postId" class="form-check-input">
                </div>
              </td>
              <td>
                <div style="padding: 10px 0;">{{ post.postId }}</div>
              </td>
              <td>
                <div style="padding: 10px 0; text-align:center;">
                  <router-link style="text-decoration: none; color: black; font-weight: bold; font-size: 16px; cursor: pointer; text-decoration: underline;" :to="`read/${post.postId}`" @mouseover="onMouseOver" @mouseleave="onMouseLeave">{{ post.postTitle }}</router-link>
                </div>
              </td>
              <td>
                <div style="padding: 10px 0;">{{ post.postCategory }}</div>
              </td>
              <td>
                <div style="padding: 10px 0;">{{ post.postRegisterDate }}</div>
              </td>
              <td>
                <div style="padding: 10px 0;">{{ post.postWriter }}</div>
              </td>
              <td>
                <div style="padding: 10px 0;">
                  <div class="dropdown">
                    <svg width="5" height="15" id="dropdownMenuButton" xmlns="http://www.w3.org/2000/svg" role="button" data-bs-toggle="dropdown" aria-expanded="false" style="pointer-events: all;">
                      <circle cx="2.5" cy="2.5" r="2" fill="black"/>
                      <circle cx="2.5" cy="7.5" r="2" fill="black"/>
                      <circle cx="2.5" cy="12.5" r="2" fill="black"/>
                    </svg>
                    <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton" style="min-width: auto;">
                      <li><router-link :to="`read/${post.postId}`" class="dropdown-item" href="#">게시물 보기
                      <span><i class="fa-solid fa-magnifying-glass" style="padding:0.5rem"></i></span>
                      </router-link></li> 
                      <li><hr class="dropdown-divider"></li>
                      <li><router-link :to="`modify/${post.postId}`" class="dropdown-item" href="#">게시물 수정
                        <span><i class="fa-solid fa-gear" style="padding:0.5rem"></i></span>
                      </router-link></li>
                      <li><hr class="dropdown-divider"></li>
                      <li><a @click="deletePost(post.postId)" class="dropdown-item" href="#">게시물 삭제
                      <span><i class="fa-solid fa-trash-can" style="padding:0.5rem"></i></span>
                      </a></li>
                    </ul>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
          <tfoot>
          </tfoot>
        </table>
      </div>
      <!-- 페이지네이션 -->
      <div class="mt-3 mb-3">
        <div class="d-flex justify-content-start">
          <button class="btn btn-dark" @click="selectAllPosts" style="color:white; margin-right: 10px;"><b>전체선택</b></button>
          <button class="btn btn-outline-dark" @click="deleteSelected"><b>삭제</b></button>
        </div>
        <div class="d-flex justify-content-center">
        <button class="btn btn-white mr-2" @click="prevPage" :disabled="currentPage === 0">이전</button>
        <div v-for="pageNumber in totalPages" :key="pageNumber">
          <button class="btn btn-white mr-2" @click="goToPage(pageNumber)" :style="{ 'font-weight': currentPage + 1 === pageNumber ? 'bold' : 'normal' }">{{ pageNumber }}</button>
        </div>
        <button class="btn btn-white" @click="nextPage" :disabled="currentPage === totalPages -1">다음</button>
      </div>
      </div>
    </div>
  </main>
</template>

<script>
// eslint-disable-next-line
/* eslint-disable */
import axios from 'axios';
export default {

  name: 'AdminList',
  data () {
    return {
      postList: [],
      currentPage: 0,
      totalPages: 0,
      itemsPerPage: 10,
      selectedCategory: '전체', // 선택된 카테고리를 저장하는 변수 추가
      // modalVisible: false, // 모달의 표시 여부
      allSelected: false, // 모든 항목이 선택되었는지 여부를 추적
      selectedPosts: [], // 선택된 게시물 ID를 저장하는 배열
    };
  },
  mounted() {
    this.axiosAdminPostList();
  },
  methods: {
    async axiosAdminPostList(category = '전체') {
      try {
        const accessToken = localStorage.getItem('accessToken')

        let url = `http://localhost:8080/api/v1/posts/post/list?page=${this.currentPage}&size=${this.itemsPerPage}`;
        if(category !== '전체') {
          url = `http://localhost:8080/api/v1/posts/post/${category}?page=${this.currentPage}&size=${this.itemsPerPage}`;
        }
        const response = await axios.get(url, {
          headers: {
            'Authorization': `Bearer ${accessToken}`
          }});
        this.postList = response.data.content;
        this.totalPages = response.data.totalPages;
      } catch (err) {
        // alert('관리자만 이용할 수 있습니다.');
        // this.$store.commit('setLoggedIn', this.loggedIn)
        // this.$router.push('/');
        console.error('리스트를 불러 올 수 없습니다.', err);
      }
    },
    async selectCategory(category) {
      this.selectedCategory = category;
      await this.axiosAdminPostList(category);
    },
    goToPage(pageNumber) {
      this.currentPage = pageNumber-1;
      this.axiosAdminPostList(this.selectedCategory); // Pass the selected category
    },
    prevPage() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.axiosAdminPostList(this.selectedCategory);
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.axiosAdminPostList(this.selectedCategory);
      }
    },
    handleCategoryChange(event) {
        const category = event.target.value;
    this.selectCategory(category);
    },
    readPost(postId) {
      // 수정할 게시물의 ID를 전달하여 수정 페이지로 이동
      this.$router.push(`/admin/post/read/${postId}`);
    },
    editPost(postId) {
      // 수정할 게시물의 ID를 전달하여 수정 페이지로 이동
      this.$router.push(`/admin/post/modify/${postId}`);
    },
    async deletePost(postId) {
      try {

        // 확인 창을 표시하고 사용자가 "확인"을 선택했는지 확인합니다.
    const confirmDelete = confirm('정말로 게시물을 삭제하시겠습니까?');
    
    if (confirmDelete) {

        const accessToken = localStorage.getItem('accessToken')

        const url = `http://localhost:8080/api/v1/posts/post/${postId}`;
        const response = await axios.delete(url, {
          headers: {
            'Authorization': `Bearer ${accessToken}`
          }
        });
        // 삭제 후 성공 메시지 출력
        alert('게시물이 성공적으로 삭제되었습니다.');
         // 삭제 후 게시물 목록을 다시 불러옴
        await this.axiosAdminPostList(this.selectedCategory);
        // 삭제 후 리스트 페이지로 리다이렉트
        this.$router.push('/admin/post/list');
        return response;
      }
      } catch (error) {
        console.error('게시물 삭제 실패:', error);
      }
    },
    // 모든 항목을 선택하거나 선택 해제합니다.
  selectAllPosts() {
  if (!this.allSelected) {
    // 전체 선택 버튼을 클릭한 경우 모든 항목을 선택합니다.
    this.selectedPosts = this.postList.map(post => post.postId);
  } else {
    // 이미 모든 항목이 선택된 상태이면 선택을 해제합니다.
    this.selectedPosts = [];
  }
  // 전체 선택 상태를 업데이트합니다.
  this.allSelected = !this.allSelected;
},
  // 선택된 항목을 삭제합니다.
  async deleteSelected() {
    // 선택된 게시물이 있는지 확인
  if (this.selectedPosts.length === 0) {
    // 선택된 게시물이 없는 경우 알림 표시
    alert('삭제할 게시물을 선택해주세요.');
    return; // 함수 종료
  }
    // 선택된 게시물이 있는 경우 삭제 확인 대화 상자 표시
    const confirmDelete = confirm('정말로 선택된 게시물을 삭제하시겠습니까?');
    if (confirmDelete) {
      try {
        const accessToken = localStorage.getItem('accessToken');
        // 선택된 게시물 ID를 반복하며 각각을 삭제합니다.
        await Promise.all(this.selectedPosts.map(async postId => {
          const url = `http://localhost:8080/api/v1/posts/post/${postId}`;
          await axios.delete(url, {
            headers: {
              'Authorization': `Bearer ${accessToken}`
            }
          });
        }));
        // 성공 메시지를 표시합니다.
        alert('선택된 게시물이 성공적으로 삭제되었습니다.');
        // 게시물 목록을 새로고침합니다.
        await this.axiosAdminPostList(this.selectedCategory);
        // 선택된 항목을 초기화합니다.
        this.selectedPosts = [];
      } catch (error) {
        console.error('게시물 삭제 실패:', error);
      }
    }
},
    onMouseOver(event) {
      event.target.style.color = 'blue'; // 마우스를 올렸을 때 색상 변경
    },
    onMouseLeave(event) {
      event.target.style.color = 'black'; // 마우스가 벗어났을 때 색상 변경
}
  },
};
</script>

<style>
.margin-bottom {
  margin-bottom: 3rem;
}

thead th {
  border-bottom: 2px solid black; /* 테두리 진하게 설정 */
  background-color: #f2f2f2; /* 배경색 설정 */
}
</style>
