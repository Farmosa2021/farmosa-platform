<template>
  <v-card class="mx-auto my-5" color="#303030" max-width="400" min-width="300">
    <v-list-item two-line>
      <v-list-item-content>
        <v-list-item-title class="text-h5">
          {{ name }}
        </v-list-item-title>
        <!-- <v-list-item-subtitle>Mon, 12:30 PM, Mostly sunny</v-list-item-subtitle> -->
      </v-list-item-content>
      <v-row align="center" justify="end" class="mx-1">
        <v-btn icon dark @click="addFav">
          <v-icon>
            {{ active ? "mdi-heart" : "mdi-heart-outline" }}
          </v-icon>
        </v-btn>
      </v-row>
    </v-list-item>

    <v-list-item>
      <v-row align="center">
        <v-col class="text-h3" cols="6"> ${{ currentPrice }} </v-col>
        <v-col cols="6">
          <v-avatar size="72">
            <v-img :src="link" width="92"
              ><div class="fill-height bottom-gradient"></div
            ></v-img>
          </v-avatar>
        </v-col>
      </v-row>
    </v-list-item>
    <v-chip color="red" text-color="white" class="ml-3 my-1" small>
      產地資訊
    </v-chip>
    <v-chip
      v-if="currentPrice > 0"
      color="green"
      text-color="white"
      class="ml-3 my-1"
      small
    >
      當季好蔬果
    </v-chip>

    <v-list-item>
      <v-list-item-icon>
        <v-icon>mdi-send</v-icon>
      </v-list-item-icon>
      <v-list-item-subtitle>{{ postNum }} posts</v-list-item-subtitle>
    </v-list-item>

    <v-divider></v-divider>

    <v-card-actions>
      <v-btn text @click="fullPage"> Full Page </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  props: {
    name: String,
  },
  mounted: async function () {
    console.log(this.name);
    const link = await this.$axios.$post("/fruits/images", {
      fruit: this.name,
    });
    console.log(link.data);
    this.link = link.data[0]["image"];

    const price = await this.$axios.$post("/fruits/history", {
      fruit: this.name,
    });
    // console.log(price.data.data_1[price.data.data_1.length-1][this.name]);
    this.currentPrice =
      price.data.data_1[price.data.data_1.length - 1][this.name].toFixed(1);
    const n = await this.$axios.$get("/posts/fruits/" + this.name);
    // console.log(n.data.length);
    this.postNum = n.data.length;

    let uid = this.$auth.$storage.getLocalStorage("uid");
    // console.log(uid);
    let fav = await this.$axios.$get("/users/" + uid + "/favor", {
      uid: uid,
    });
    if (fav.result != "error") {
      let f = [];
      for (let i = 0; i < fav.data.length; i++) {
        if (fav.data[i].fruit == this.name) {
          this.active = true;
        }
      }
    }
    // console.log(fav.data);
  },
  data: () => ({
    link: "",
    season: true,
    postNum: 0,
    currentPrice: 0,
    active: false,
  }),
  methods: {
    async addFav() {
      let uid = this.$auth.$storage.getLocalStorage("uid");
      if (this.active == true) {
        await this.$axios.$delete("/users/" + uid + "/favor", {
          data: {
            uid: uid,
            fruit: this.name,
          },
        });
      } else {
        await this.$axios.$post("/users/" + uid + "/favor", {
          uid: uid,
          fruit: this.name,
        });
      }
      this.active = !this.active;
    },
    async fullPage() {
      this.$router.push("/fruits/" + this.name);
    },
  },
};
</script>

