<template>
  <div>
    <v-row>
      <v-avatar size="102">
        <v-img :src="link" width="92"
          ><div class="fill-height bottom-gradient"></div
        ></v-img>
      </v-avatar>
      <h1 class="my-auto mx-5">{{ this.fruit }}</h1>
      <v-btn class="my-auto" icon dark @click="addFav">
        <v-icon large>
          {{ active ? "mdi-heart" : "mdi-heart-outline" }}
        </v-icon>
      </v-btn>
    </v-row>
    <v-row>
      <v-col>
        <price v-if="histExist" :name="this.fruit" />
        <stac-price v-if="histExist" :name="this.fruit" />
        <now-price v-if="nowExist" :name="this.fruit" />
        <now-market v-if="nowExist" :name="this.fruit" />
      </v-col>
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
        <post v-for="i in this.pids" :key="i" :pid="i" :color="colors[i%3]" />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Post from "~/components/Post.vue";
import NowMarket from "../../components/NowMarket.vue";
import Price from "../../components/Price.vue";

export default {
  middleware: ["auth"],
  components: {
    Post,
    Price,
    NowMarket,
  },
  async asyncData({ params }) {
    const fruit = params.fruit; // When calling /abc the user will be "abc"
    return { fruit };
  },
  data() {
    return {
      pids: [],
      link: "",
      histExist: false,
      nowExist: false,
      fruitV: [],
      expand: false,
      title: "",
      content: "",
      success: false,
      error: false,
      active: false,
      colors: ['#385F73', '#1F7087', '#00838F'],
    };
  },
  methods: {
    async newPost() {
      let suc = await this.$axios.$post("/posts/", {
        author: this.$auth.$storage.getLocalStorage("user"),
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
      this.title = "";
      this.content = "";
    },
    async sleep(ms = 0) {
      return new Promise((r) => setTimeout(r, ms));
    },
    async getPids() {
      const pids = await this.$axios.$get("/posts/fruits/" + this.fruit);
      // console.log(pids);
      var p = [];
      for (let i = pids.data.length - 1; i >= 0; i--) {
        p.push(pids.data[i]["PID"]);
      }
      this.pids = p;
    },
    async addFav() {
      let uid = this.$auth.$storage.getLocalStorage("uid");
      if (this.active == true) {
        await this.$axios.$delete("/users/" + uid + "/favor", {
          data: {
            uid: uid,
            fruit: this.fruit,
          },
        });
      } else {
        await this.$axios.$post("/users/" + uid + "/favor", {
          uid: uid,
          fruit: this.fruit,
        });
      }
      this.active = !this.active;
    },
  },
  mounted: async function () {
    this.getPids();
    let uid = this.$auth.$storage.getLocalStorage("uid");
    // console.log(uid);
    let fav = await this.$axios.$get("/users/" + uid + "/favor", {
      uid: uid,
    });
    console.log(fav);
    if (fav.result != "error") {
      for (let i = 0; i < fav.data.length; i++) {
        if (fav.data[i].fruit == this.fruit) {
          this.active = true;
          break;
        }
      }
    }
    const link = await this.$axios.$post("/fruits/images", {
      fruit: this.fruit,
    });
    // console.log(link)
    if (link.result != "error") {
      this.histExist = true;
      // console.log(link.data[0]["image"]);
      this.link = link.data[0]["image"];
    }
    const m = await this.$axios.$get("/fruits/realtime/markets");
    let mar = [];
    for (let i = 0; i < m.data.length; i++) {
      mar.push(m.data[i]["market"]);
    }
    for (let i = 0; i < mar.length; i++) {
      const real = await this.$axios.$post("/fruits/realtime", {
        fruit: this.fruit,
        market: mar[i],
      });
      if (real.result != "no result") {
        this.nowExist = true;
        break;
      }
    }
  },
};
</script>