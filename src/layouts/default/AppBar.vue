<template>
  <v-app-bar :elevation="5">
    <v-app-bar-title class="d-flex">
      <div @click="$router.push('/')" style="cursor: pointer;">
        <v-icon color="tiktokRed" icon="mdi-music-circle-outline" />
        <span class="ml-2" :style="{ color: $vuetify.theme.themes.dark.colors.tiktokBlue }">TokTik</span>
      </div>
    </v-app-bar-title>
    <template v-slot:append>
      <v-btn @click="toggleTheme" class="mr-2" icon="mdi-theme-light-dark" />

      <!-- Notifications Menu -->
      <v-menu transition="scale-transition" width="420" location="bottom center" :close-on-content-click="false">
        <template v-slot:activator="{ props: notifMenu }">
          <v-btn v-bind="notifMenu" class="mr-2" icon="mdi-bell-outline" />
        </template>

        <v-card v-if="notifs.length === 0" variant="elevated" class="mb-4 pa-6">
          <h3 class="text-center mb-6">No notifications.</h3>
          <p v-if="!isLoggedIn" class="text-center">Log in to receive notifications.</p>
        </v-card>
        <v-virtual-scroll :items="notifs" height="450">
          <template v-slot:default="{ item }">
            <v-card>
              <div class="d-flex align-center pr-4" style="gap: 15px;">
                <v-card @click="playVideo(item.videoKey)" class="pa-2 flex-grow-1">
                  <!-- <v-card-title>{{ item.title }}</v-card-title> -->
                  <v-card-title>{{ item.message }}</v-card-title>
                  <v-card-subtitle>Click to view</v-card-subtitle>
                </v-card>
                <div @click="deleteNotif(item.id)" variant="tonal" >
                  <v-btn icon="mdi-delete"></v-btn>
                </div>
              </div>
            </v-card>
          </template>

        </v-virtual-scroll>

      </v-menu>


      <v-btn @click="$router.push('/Upload')" class="mr-2" prepend-icon="mdi-plus">Upload</v-btn>
      <template v-if="isLoggedIn">
        <v-menu open-on-hover>
          <template v-slot:activator="{ props }">
            <v-btn append-icon="mdi-account" v-bind="props" style="text-transform: unset !important;">{{ username
            }}</v-btn>
          </template>

          <v-list>
            <v-list-item v-for="(item, i) in items" :key="i" @click="item.event">
              <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
      <v-btn v-if="!isLoggedIn" variant="outlined" @click="login">Log in</v-btn>
    </template>
  </v-app-bar>
</template>

<script lang="ts">
import { useAppStore } from '@/store/app'
import { useTheme } from 'vuetify'
import { socket, joinRoom, leaveRoom } from "@/socket";
const store = useAppStore()

export default {
  data: () => ({
    isLoggedIn: false,
    username: store.username,
    items: [],
    notifs: [
      //  { title: "Title", message: "Someone liked your video." },
      //  { title: "Title", message: "Someone liked your video." },
      //  { title: "Title", message: "Someone liked your video." },
      //  { title: "Title", message: "Someone liked your video." },
      //  { title: "Title", message: "Someone liked your video." },
    ],
    pageRoom: "navbar",
  }),
  setup() {
    const theme = useTheme()
    return {
      theme
    }
  },
  mounted() {
    this.items = [
      { title: "View Profile", event: this.viewProfile },
      { title: "Log out", event: this.logout }
    ]
    this.syncLogin()
    this.getNotifs()

    joinRoom(this.pageRoom)
    socket.on('notifUpdate', async () => {
      if (!store.isLoggedIn) {
        return;
      }
      console.log('Getting notifs')
      this.getNotifs()
    })

  },
  methods: {
    playVideo(key: String) {
      this.$router.push({ name: 'PlayVideo', params: { key: key as any }, query: { key: key as any } });
    },
    async getNotifs() {
      if (!store.isLoggedIn) {
        return
      }
      try {
        const res = await this.axios.get('/api/notifs?username=' + store.username)
        console.log(res.data);
        this.notifs = res.data
      } catch (e) {

      }
    },
    async deleteNotif(notifId) {
      try {
        const res = await this.axios.delete('/api/notif?username=' + store.username + '&notifId=' + notifId)
        console.log(res.data);
        this.notifs = res.data
      } catch (e) {

      }
    },
    syncLogin() {
      this.isLoggedIn = store.isLoggedIn
    },
    logout() {
      store.isLoggedIn = false
      this.syncLogin()
      this.$router.push('/Login')
    },
    login() {
      if (store.isLoggedIn) {
        return
      }
      this.$router.push('/Login')
    },
    viewProfile() {
      this.$router.push('/Profile/' + this.parseJwt(localStorage.getItem('jwt')).username)
    },
    parseJwt(token) {
      var base64Url = token.split('.')[1];
      var base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
      var jsonPayload = decodeURIComponent(window.atob(base64).split('').map(function (c) {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
      }).join(''));
      console.log(JSON.parse(jsonPayload))
      return JSON.parse(jsonPayload);
    },
    toggleTheme() {
      this.theme.global.name.value = this.theme.global.current.value.dark ? 'light' : 'dark'
    }
  },
}

</script>
