<template>
  <v-dialog v-model="dialog" persistent max-width="600px">
    <template v-slot:activator="{ on, attrs }">
      <v-btn dark v-bind="attrs" v-on="on" plain x-large class="text-h5">
        LOGIN
      </v-btn>
    </template>
    <v-card>
      <v-alert type="error" :value="error" transition="fade-transition"> Error. </v-alert>
      <v-alert type="success" :value="success" transition="fade-transition"> Success. </v-alert>
      <v-card-title>
        <span class="text-h5">User Profile</span>
      </v-card-title>
      <v-card-text>
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-text-field
                label="Name*"
                required
                v-model="username"
              ></v-text-field>
            </v-col>
            <v-col cols="12">
              <v-text-field
                label="Password*"
                type="password"
                required
                v-model="password"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-container>
        <small>*indicates required field</small>
      </v-card-text>
      <v-card-actions>
        <v-btn color="blue darken-1" text @click="register"> REGISTER </v-btn>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click="dialog = false">
          Close
        </v-btn>
        <v-btn color="blue darken-1" text @click="login"> LOGIN </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data: () => ({
    dialog: false,
    username: "",
    password: "",
    error: false,
    success: false,
  }),
  methods: {
    async login() {
      // let suc = await this.$axios.$post("/users/auth", {
      //   username: this.username,
      //   password: this.password,
      // });
      let suc = await this.$auth.loginWith('local', {
          data: {
            username: this.username,
            password: this.password,
          }
      });
      console.log(suc);
      if(suc.data.result=='success'){
      // if(this.$auth.loggedIn){
        this.success = true;
        this.$auth.setUser(this.username);
        this.$auth.$storage.setLocalStorage('user', this.username, true);
        this.$auth.$storage.setLocalStorage('uid', suc.data.data[0]["UID"], true);
        console.log(this.$auth.$storage.getLocalStorage('user'));
        console.log(this.$auth.$storage.getLocalStorage('uid'));
        await this.sleep(1000);
        this.success = false;
        // this.$router.push('/users/' + this.username);
        this.$router.push('/search');
      }else{
        this.error = true;
        await this.sleep(1000);
        this.error = false;
        this.password = ''
      }
    },
    async register() {
      let suc = await this.$axios.$post("/users/", {
        username: this.username,
        password: this.password,
      });
      if(suc.result=='success'){
        this.success = true;
        await this.sleep(1000);
        this.success = false;
        // this.$router.push('/users/' + this.username);
      }else{
        this.error = true;
        await this.sleep(1000);
        this.error = false;
      }
    },
    async sleep(ms = 0) {
      return new Promise(r => setTimeout(r, ms));
    }
  },
};
</script>
