<template>
  <div class="max-w-prose mx-auto">
    <section class="p-4">
      <section class="flex flex-row items-center mb-2 p-2 border rounded  focus-within:border-slate-400">
        <label for="search-input">
          <svg width="20" height="20" class="DocSearch-Search-Icon" viewBox="0 0 20 20"><path d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z" stroke="currentColor" fill="none" fill-rule="evenodd" stroke-linecap="round" stroke-linejoin="round"></path></svg>
        </label>
        <input id="search-input" class="w-full ml-2 focus:outline-0" v-model="usersName" type="search" placeholder="Search user" />
      </section>
      <div class="flex flex-row items-center mx-4">
        <label for="genderAny" class="mr-3">
          <input type="radio" name="gender" id="genderAny" v-model="usersGender" value="">
          <span class="ml-1">Any</span>
        </label>
        <label for="genderMale" class="mr-3">
          <input type="radio" name="gender" id="genderMale" v-model="usersGender" value="male">
          <span class="ml-1">Male</span>
        </label>
        <label for="genderFemale" class="mr-3">
          <input type="radio" name="gender" id="genderFemale" v-model="usersGender" value="female">
          <span class="ml-1">Female</span>
        </label>
      </div>
    </section>
    <infinite-scroll
        @infinite-scroll="getUsers"
        :message="message"
        :noResult="noResult"
      >
      <ul class="divide-y divide-gray-100" v-if="filteredUsers.length > 0">
        <li v-for="(u, i) in filteredUsers" :key="i">
          <user-brief 
            :thumbnail="u.picture.thumbnail"
            :name="u.name"
            :email="u.email"
            :selectedUser="selectedUser"
            @toggle-user="toggleUser(`${u.name.last}-${u.name.first}`)"
          />
          <user-details
            v-if="`${u.name.last}-${u.name.first}` === selectedUser"
            :id="u.id"
            :name="u.name"
            :picture="u.picture.large"
            :location="u.location"
            :phone="u.phone"
            :cell="u.cell"
          />
        </li>
      </ul>
      <p v-else>No results to display.</p>
      <button class="w-8/12 block rounded border p-2 mx-auto my-4" @click="getUsers">Load More</button>
    </infinite-scroll>
  </div>
</template>

<script>
const RESULTS_AMOUNT = 25

import axios from 'axios'
import InfiniteScroll from 'infinite-loading-vue3'

import UserDetails from './components/UserDetails'
import UserBrief from './components/UserBrief'

export default {
  components: {
    InfiniteScroll,
    UserDetails,
    UserBrief
  },
  
  name: 'App',

  data() {
    return {
      users: [],
      usersName: '',
      usersGender: '',
      selectedUser: null,
      trendingRepos: [],
      noResult: false,
      message: "",
      RESULTS_AMOUNT
    }
  },

  mounted() {
    if (this.localData !== null) {
      this.users = JSON.parse(this.localData)
      this.usersName = this.paramsData.name || ''
      this.usersGender = this.paramsData.gender || ''
      this.selectedUser = this.paramsData.user || null
    } else {
      this.getUsers()
    }
  },

  computed: {
    filteredUsers() {
      return this.users.filter(user => {
        const nameToLowercase = this.usersName.toLowerCase()
        const genderToLowercase = this.usersGender.toLowerCase()

        const isFirstName = user.name.first.toLowerCase().includes(nameToLowercase)
        const isLastName = user.name.last.toLowerCase().includes(nameToLowercase)
        const isGender = genderToLowercase === '' || user.gender.toLowerCase() === genderToLowercase

        return  isGender && (isFirstName || isLastName)
      })
    },

    localData() {
      return localStorage.getItem('users-list', JSON.stringify(this.users))
    },

    paramsData() {
      return Object.fromEntries(new URL(window.location).searchParams.entries())
    },

    pageNumber() {
      return parseInt(this.users.length/RESULTS_AMOUNT)
    },

    winParams() {
      return {
        name: this.usersName,
        gender: this.usersGender,
        page: this.pageNumber,
        user: this.selectedUser
      }
    }
  },

  methods: {
    async getUsers() {
      const url = `https://randomuser.me/api/?page=${this.pageNumber}&results=${this.RESULTS_AMOUNT}`
      try {
        const result = await axios.get(url)
        if(result.data.results.length) {
          this.users.push(...result.data.results)
          localStorage.setItem('users-list', JSON.stringify(this.users))
        } else {
          this.noResult = true;
          this.message = "No result found";
        }
      } catch(error) {
        this.noResult = true;
        this.message = `Error loading data: ${error}`;
        localStorage.removeItem('users-list')
      }
    },

    resetFilter() {
      this.usersName = ''
      this.usersGender = ''
    },

    toggleUser(name) {
      if (this.selectedUser === name) {
        this.selectedUser = null
      } else {
        this.selectedUser = name
      }
    },

    updateParams() {
      const params = this.winParams
      const query = Object.keys(params).map(key => params[key] ? key + '=' + params[key] : '').filter(n => n).join('&');
      const url = query ? `${window.location.pathname}?${query}` : `${window.location.pathname}`

      window.history.pushState(null, document.title, url)
    }
  },

  watch: {
    winParams() {
      this.updateParams()
    }
  }
};
</script>

<style>
.gender {
  position: absolute;
  visibility: hidden;
}
.details {
  position: relative;
  top: 1px;
}
</style>
