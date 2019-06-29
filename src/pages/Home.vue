<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">
        <div class="error" v-if="error">{{ error }}</div>
        <search 
          :value="search" 
          placeholder="Type username..."
          @search="search = $event" />
        <button @click="getRepos" class="btn btnPrimary">Search</button>

        <div class="content__wrapper" v-if="user">
          <div class="user_info" v-if="user">
            <p>{{ user.name }}</p>
            <img :src="user.avatar_url" :alt="user.name">
            <p>Repositories: {{user.public_repos}}</p>
          </div>

          <div class="repos__wrapper" v-if="repos">
            <div class="repos__sort-wrap">
              <p @click="sort('name')">Name &#8595;</p>
              <p @click="sort('stargazers_count')">Stars &#8595;</p>
            </div>
            
            <div class="repos__item" v-for="repo in reposSort" :key="repo.id">
              <div class="repos-info">
                <a class="link" target="_blank" :href="repo.html_url">{{repo.name}}</a>
                <span>{{repo.stargazers_count}} ⭐</span>
              </div>
            </div>

            <div class="repos_pagination">
              <div class="btn btnPrimary" @click="prevPage"> &#8592;</div>
              <div class="btn btnPrimary" @click="nextPage"> &#8594;</div>
            </div>
          </div>
        </div>        
        
      </div>
    </section>
  </div>
</template>

<script>
import search from '@/components/Search.vue';
import axios from 'axios';

export default {
  components: { search },
  data() {
    return {
      search: '',
      repos: null,
      user: null,
      error: null,
      currentSort: 'name',
      currentSortDir: 'asc',
      page: {
        current: 1,
        length: 4
      }
    }
  },
  mounted() {
    document.body.addEventListener('keydown', (e) => {
      if (e.keyCode === 13) {
        this.getRepos();
      }
    })
  },
  computed: {
    reposSort () {
      return this.repos.sort((a, b) => {
        let mod = 1
        if (this.currentSortDir === 'desc') mod = -1
        if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
        if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
        return 0
      }).filter((row, index) => {
        let start = (this.page.current-1) * this.page.length
        let end = this.page.current * this.page.length
        if (index >= start && index < end) return true
      })
    }
  },
  methods: {
    getRepos() {
      axios
        .get(`https://api.github.com/users/${this.search}`)
        .then(res => {
          this.user = res.data;
          this.error = null;
          axios.get(`https://api.github.com/users/${res.data.login}/repos`)
          .then(response => {
            this.repos = response.data;
          });
        })
        .catch(err => {
          this.repos = null;
          this.user = null;
          this.error = `Cant't find this user`;
        })
    },
     sort (e) {
      if (e === this.currentSort) {
        this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.currentSort = e;
    },
    // Pagination
    prevPage () {
      if (this.page.current > 1) this.page.current-=1
    },
    nextPage () {
      if ((this.page.current * this.page.length) < this.repos.length) this.page.current+=1
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
.content__wrapper {
  align-items: flex-start;
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.user_info {
  img {
    width: 200px;
    border-radius: 10px;
    margin-right: 20px;
  }
  p {
    font-size: 20px;
    margin: 10px 0;
  }
}
button {
  margin-top: 20px;
}
.repos__wrapper {
  margin: 30px 0;
}
.repos__sort-wrap {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-weight: bold;
  cursor: pointer;
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}
.repos_pagination {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>


