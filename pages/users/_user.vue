<template>
  <div>
    <h1>{{ this.$auth.$storage.getLocalStorage("user") }}</h1>
    <!-- <new-post :author="this.user"/> -->
    <v-row>
      <v-col>
        <v-btn plain @click="expand = !expand"> NEW POST </v-btn>
        <v-expand-transition>
          <v-card
            class="mx-auto my-5"
            color="#385F73"
            dark
            max-width="500"
            v-show="expand"
          >
            <v-alert type="error" :value="error" transition="fade-transition">
              Error.
            </v-alert>
            <v-alert
              type="success"
              :value="success"
              transition="fade-transition"
            >
              Success.
            </v-alert>
            <!-- <v-card-title>
          <span class="text-h5">New Post</span>
        </v-card-title> -->
            <v-card-text
              ><v-form>
                <v-text-field
                  label="Title"
                  single-line
                  full-width
                  hide-details
                  v-model="title"
                ></v-text-field>
                <v-divider></v-divider>
                <v-text-field
                  label="Fruits"
                  single-line
                  full-width
                  hide-details
                  v-model="fruit"
                ></v-text-field>
                <v-divider></v-divider>
                <v-textarea
                  label="Content"
                  counter
                  maxlength="120"
                  full-width
                  single-line
                  v-model="content"
                ></v-textarea>
              </v-form>
              <small>*indicates required field</small>
            </v-card-text>
            <v-card-actions>
              <v-btn text @click="expand = !expand"> Close </v-btn>
              <v-spacer></v-spacer>
              <v-btn text @click="newPost"> Post </v-btn>
            </v-card-actions>
          </v-card>
        </v-expand-transition>
        <post v-for="i in this.pids" :key="i" :pid="i" :color="colors[i%3]"/>
      </v-col>
      <v-col>
        <fruit-info v-for="i in this.histFav" :key="i" :name="i" />
        <now-info v-for="i in this.nowFav" :key="i" :name="i"/>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Post from "~/components/Post.vue";
import FruitInfo from "../../components/FruitInfo.vue";
import NewPost from "../../components/NewPost.vue";

export default {
  middleware: ["auth"],
  components: {
    Post,
    NewPost,
    FruitInfo,
  },
  // async asyncData({ params }) {
  //   const user = params.user; // When calling /abc the user will be "abc"
  //   return { user };
  // },
  data() {
    return {
      pids: [],
      expand: false,
      fruit: "",
      title: "",
      content: "",
      success: false,
      error: false,
      user: "haha",
      nowFav: [],
      histFav: [],
      colors: ['#385F73', '#1F7087', '#00838F'],
    };
  },
  mounted: async function () {
    this.user = this.$auth.$storage.getLocalStorage("user");
    let uid = this.$auth.$storage.getLocalStorage("uid");
    // console.log(this.user);
    const pids = await this.$axios.$get("/posts/users/" + this.user);
    // console.log(pids.data[0]["PID"]);
    var p = [];
    for (let i = pids.data.length - 1; i >= 0; i--) {
      p.push(pids.data[i]["PID"]);
    }
    this.pids = p;

    let fav = await this.$axios.$get("/users/" + uid + "/favor", {
      uid: uid,
    });
    if (fav.result != "error") {
      let f = [];
      for (let i = 0; i < fav.data.length; i++) {
        f.push(fav.data[i]["fruit"]);
      }
      // console.log(f);
      for (let i = 0; i < f.length; i++) {
        const link = await this.$axios.$post("/fruits/images", {
          fruit: f[i],
        });
        if (link.result != "error") {
          this.histFav.push(f[i]);
        } else {
          this.nowFav.push(f[i]);
        }
      }
    }
  },
  methods: {
    async getPids() {
      const pids = await this.$axios.$get("/posts/users/" + this.user);
      // console.log(pids.data[0]["PID"]);
      var p = [];
      for (let i = pids.data.length - 1; i >= 0; i--) {
        p.push(pids.data[i]["PID"]);
      }
      this.pids = p;
    },
    async newPost() {
      let suc = await this.$axios.$post("/posts/", {
        author: this.user,
        title: this.title,
        fruit: this.fruit,
        content: this.content,
      });
      console.log(suc);
      if (suc.result == "success") {
        this.success = true;
        await this.sleep(1000);
        this.success = false;
        // this.$router.push('/users/' + this.author);
        this.expand = !this.expand;
        this.getPids();
      } else {
        this.error = true;
        await this.sleep(1000);
        this.error = false;
        this.expand = !this.expand;
      }
      this.fruit = "";
      this.title = "";
      this.content = "";
    },
    async sleep(ms = 0) {
      return new Promise((r) => setTimeout(r, ms));
    },
  },
};
</script>