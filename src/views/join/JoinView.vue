<template>
<!-- // eslint-disable-next-line
/* eslint-disable */ -->
  <main class="main-content">
    <div class="container">
      <!-- 1단계 -->
      <div class="joinBox" v-if="currentStep === 1">
        <h4 class="mb-3" style="line-height:1.4;">약관에 동의해주세요.</h4>

        <div class="mb-4">
          <div class="col">
            <input type="checkbox" v-model="agreeAll" id="agree-all" style="margin-right:0.4rem;" @change="toggleAllAgreements" />
            <label class="mb-1" for="agree-all" style="font-size:1.1rem;"><b>모두 동의</b></label>
          </div>

          <div class="col">
            <input type="checkbox" v-model="age" id="agree-age" style="margin-right:0.4rem;" @change="toggleAge" />
            <label class="form-check-label" for="agree-age">[필수] 만 14세 이상입니다.</label>
          </div>

          <div class="col">
            <input type="checkbox" v-model="policy" id="agree-policy" style="margin-right:0.4rem;" @change="togglePolicy" />
            <label class="form-check-label" for="agree-policy">[필수] 누림 약관 및 동의사항  <router-link to="#">보기</router-link></label>
          </div>
        </div>

        <div class="col">
            <button type="button" class="btn btn-primary" @click="goNextStep">동의하고 가입하기</button>
          </div>
      </div>

      <!-- 2단계 -->
      <!-- <div class="joinBox" v-if="currentStep === 2">
        <h4>회원 유형을 선택해주세요.</h4>

        <div class="mb-3">
          <div class="col" style="margin-bottom:0.6rem; border-radius:1rem; border:1px solid #E0E0E0; padding:1rem;">
            <input type="radio" v-model="type" value="0" id="normal-member" />
            <label for="agree-age">일반회원</label><br>
            <span class="form-check-label">신혼부부 지원정책에 대해 알고싶어요.</span>
          </div>

          <div class="col" style="border-radius:1rem; border:1px solid #E0E0E0; padding:1rem;">
            <input type="radio" v-model="type" value="1" id="expert-member" />
            <label for="agree-age">전문가 회원</label><br>
            <span class="form-check-label">누림의 멘토가 되고 싶어요.</span>
          </div>
        </div>

        <div class="col">
          <button type="button" class="btn btn-secondary" @click="goPreviousStep">이전</button>
          <button type="button" class="btn btn-primary" @click="goNextStep">다음</button>
        </div>
      </div> -->

      <!-- 3단계 -->
      <div class="joinBox" v-if="currentStep === 2">
        <h4 class="mb-3" style="line-height:1.4;">로그인에 사용할 아이디와 <br> 비밀번호, 닉네임을 입력해주세요.</h4>

        <form class="mb-5">
          <!-- Email input -->
          <div class="form-group first mb-1">
            <label class="form-check-label">이메일 주소</label>
            <input type="email" class="form-control" v-model="memberEmail" />
          </div>

          <!-- Password input -->
          <div data-mdb-input-init class="form-outline mb-1">
            <label class="form-check-label">비밀번호</label>
            <input type="password" v-model="memberPw" class="form-control" />
          </div>

          <!-- Nickname input -->
          <div data-mdb-input-init class="form-outline mb-4">
            <label class="form-check-label">닉네임</label>
            <input type="text" v-model="memberNickname" class="form-control" />
          </div>
        </form>

        <div class="col">
          <button type="button" class="btn btn-secondary me-1" @click="goPreviousStep">이전</button>
          <button type="button" class="btn btn-primary" @click="registerMember">가입하기</button>
        </div>

      </div>

    </div>
    <!-- container end -->
  </main>
</template>

<script>
// eslint-disable-next-line
/* eslint-disable */
import axios from 'axios'

export default {
  name: '',
  components: {},
  data () {
    return {
      agreeAll: true,
      policy: true,
      age: true,
      type: '0',
      currentStep: 1,
      enableSubmit: false
    }
  },
  methods: {
    goPreviousStep () {
      this.currentStep--
    },
    goNextStep () {
      if(!this.agreeAll) {
        alert('약관에 동의해주세요.')
        return
      }
      this.currentStep++
    },
    toggleAllAgreements () {
      if (this.agreeAll) {
        this.policy = true,
        this.age = true
      } else {
        this.policy = false,
        this.age = false
      }
    },
    toggleAge () {
      if (!this.age) {
        this.agreeAll = false
      }
    },
    togglePolicy () {
      if (!this.policy) {
        this.agreeAll = false
      }
    },
    async registerMember () {
      try {
        const response = await axios.post('/api/v1/members/user', {
          memberEmail: this.memberEmail,
          memberPw: this.memberPw,
          memberNickname: this.memberNickname
        })

        // Check if any of the required fields are empty
        if (!this.memberEmail || !this.memberPw || !this.memberNickname) {
          alert('회원정보를 입력해주세요.')
          return

        } else {
          console.log('회원가입 성공', response.data)
          alert('누림플러스에 오신 것을 환영합니다.')
          this.$router.push('/login')
        }

      } catch (error) {
        console.error('회원가입 실패', error)
      }
    }
  }
}
</script>

<style scoped>
.main-content {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 8em 0em 12em 0em;
}

.container {
  height: 100%;
}

.joinBox {
  min-width: 330px;
  max-width: 30%;
  margin: auto;
  text-align: left;
}

input {
  margin-right: 0.3rem;
}

.form-check-label {
  font-size: 0.9rem;
}

h4 {
  line-height: 1.6rem;
}

.span {
  font-size: 0.9rem;
}

button {
  border-radius: 2rem !important;
  padding: 0.6rem 1.2rem !important;
}
</style>
