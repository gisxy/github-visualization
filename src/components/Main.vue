<template>
  <div class="main">
    <!-- usernam input form -->
    <v-layout row wrap>
      <v-flex md6 offset-md3>
        <div class="flex-row">
          <v-text-field label="github usename" v-model="username" class="input-group--focused"></v-text-field>
          <v-btn @click="showProjects" :loading="loading" :disabled="loading">
            show</v-btn>
        </div>
      </v-flex>
    </v-layout>
    <v-layout row wrap>

      <!-- action list -->
      <v-flex md6 offset-md3 xs12>
        <v-card style="width: 100%; padding: 16px;">
          <div style="display: flex; flex-direction: column; margin-top: -32px;">
            <h3 style="margin-top: 20px;">
              <strong>Hint: username is the last part of your github profile page's URL:</strong>
              <br/>
              <strong>eg: https://github.com/ssthouse ==> ssthouse</strong>
            </h3>
          </div>
        </v-card>
      </v-flex>

      <!-- avatar -->
      <v-flex md4 offset-md4 xs12 style="margin-top: 20px; margin-bottom: 20px;">
        <v-avatar :tile="false" size="120px" class="grey lighten-4">
          <img :src="avatarUrl" alt="avatar">
        </v-avatar>
      </v-flex>
    </v-layout>

    <!-- follwing user list -->
    <users-card :userList="followingUserList" :loading="selectUserLoading" @selectUser="selectUser"></users-card>

    <v-btn-toggle v-model="viewMode" style="margin-top: 32px;">
      <v-btn flat value="2D">2D Mode</v-btn>
      <v-btn flat value="3D">3D Mode</v-btn>
    </v-btn-toggle>

    <div>
      <!-- user's project view -->
      <project-view v-show="!use3D" :repositoryList="repositoryList"></project-view>
      <!-- github view in 3d -->
      <github-view-3d v-show="use3D" :visible="use3D" :repositoryList="repositoryList3D"></github-view-3d>
    </div>

    <!-- snackbar -->
    <v-snackbar v-model="snackbar" :timeout="2000">
      {{ snackbarText }}
    </v-snackbar>
  </div>
</template>

<script>
import ProjectView from './ProjectView'
import githubView3D from './3dGithubView'
import UsersCard from './UsersCard'
import ProjectDao from './dao/projectDao'
import userRecorder from './dao/userRecorder'
import env from './util/env'
import MockData from './util/mockData'

export default {
  name: 'Main',
  components: {
    'project-view': ProjectView,
    'github-view-3d': githubView3D,
    'users-card': UsersCard
  },
  data() {
    return {
      viewMode: '3D',
      projectDao: new ProjectDao(),
      username: '',
      userRecorder,
      repositoryList: [],
      repositoryList3D: [],
      loading: false,
      selectUserLoading: false,
      snackbar: false,
      snackbarText: 'text'
    }
  },
  computed: {
    avatarUrl() {
      return this.$store.state.userinfo.avatarUrl
    },
    followingUserList() {
      return this.$store.state.userinfo.followingUserList
    },
    use3D() {
      return this.viewMode === '3D'
    }
  },
  methods: {
    showProjects() {
      this.loading = true
      this.updateUrl(this.username)
      this.$store.commit('updateUsername', this.username)
      this.projectDao.getAllProjects().then(
        () => {
          this.loading = false
          this.showSnackbar('success loading your github repo ~')
        },
        err => {
          console.log('fail' + err)
          this.loading = false
          this.showSnackbar('failed to fetch data, please try again~')
        }
      )
      this.userRecorder.addRecord(this.username)
    },
    showSnackbar(text) {
      this.snackbar = true
      this.snackbarText = text
    },
    selectUser(username) {
      this.selectUserLoading = true
      this.projectDao
        .loadUserRepositoryList(username)
        .then(repositoryList => {
          this.repositoryList = JSON.parse(JSON.stringify(repositoryList))
          this.repositoryList3D = JSON.parse(JSON.stringify(repositoryList))
          this.selectUserLoading = false
        })
        .catch(error => {
          console.log(error)
          this.selectUserLoading = false
        })
    },
    updateUrl(username) {
      this.$router.push({ name: 'main', query: { user: username } })
    },
    initUsernameFromUrl() {
      if (
        !this.$router.currentRoute.query ||
        !this.$router.currentRoute.query.user
      ) {
        return
      }
      const userInUrl = this.$router.currentRoute.query.user
      this.username = userInUrl
    }
  },
  watch: {
    '$store.state.userinfo.repositoryBeanList': {
      handler: function(newVal) {
        this.repositoryList = JSON.parse(JSON.stringify(newVal))
        this.repositoryList3D = JSON.parse(JSON.stringify(newVal))
      },
      deep: true
    }
  },
  mounted() {
    if (this.username) {
      this.showProjects()
    }
    // is dev mode ? set mock data
    if (env.isDevMode()) {
      this.repositoryList = MockData.getRepositoryList()
      this.repositoryList3D = MockData.getRepositoryList()
    }
  },
  created() {
    env.setEnv(process.env)
    this.initUsernameFromUrl()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.main {
  .flex-row {
    display: flex;
    flex-direction: row;
  }

  .action-title {
    font-size: 24px;
  }

  .card {
    padding: 16px;
  }
}
</style>
