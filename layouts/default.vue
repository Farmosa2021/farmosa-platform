<template>
  <v-app dark>
    <v-main>
      <v-card>
        <v-app-bar
          elevate-on-scroll
          scroll-target="#scrolling-techniques-7"
          color="transparent"
        >
          <v-app-bar-nav-icon></v-app-bar-nav-icon>
          <v-btn plain @click="logout"> LOG OUT </v-btn>

          <v-spacer></v-spacer>

          <v-btn icon @click="toSearch">
            <v-icon>mdi-magnify</v-icon>
          </v-btn>

          <v-btn icon @click="toHome">
            <v-icon>mdi-home</v-icon>
          </v-btn>

          <v-btn icon>
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </v-app-bar>
        <v-sheet
          id="scrolling-techniques-7"
          class="overflow-y-auto"
        >
          <v-container> 
            <nuxt/>
          </v-container>
        </v-sheet>
      </v-card>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: "Farmosa",
    };
  },
  methods: {
    async toHome() {
      if(this.$auth.$storage.getLocalStorage('user')){
        this.$router.push('/users/' + this.$auth.$storage.getLocalStorage('user'));
      }else{
        this.$router.push('/');
      }
    },
    async toSearch() {
      if(this.$auth.$storage.getLocalStorage('user')){
        this.$router.push('/search');
      }else{
        this.$router.push('/');
      }
    },
    async logout() {
      this.$auth.$storage.removeLocalStorage('user');
      this.$auth.$storage.removeLocalStorage('uid');
      this.$auth.logout();
      this.$router.push('/');
    }
  }
};
</script>
