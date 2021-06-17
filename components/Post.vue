<template>
  <v-card class="mx-auto my-5" :color="this.color" dark max-width="500">
    <v-card-title>
      <v-icon large left> mdi-apple </v-icon>
      <span class="text-h6 font-weight-bold">{{ this.title }}</span>
      <!-- <span class="text-h6 font-weight-bold">{{ this.pid }}</span> -->
    </v-card-title>

    <v-card-text class="text-h6 font-weight-bold">
      {{ content }}
    </v-card-text>
    <v-card-text>
      <v-btn outlined rounded small @click="toFruit">{{this.fruit}}</v-btn>
    </v-card-text>

    <v-card-actions>
      <v-list-item class="grow">
        <v-list-item-avatar color="grey darken-3">
          <v-img
            class="elevation-6"
            alt=""
            src="https://avataaars.io/?avatarStyle=Transparent&topType=ShortHairShortCurly&accessoriesType=Prescription02&hairColor=Black&facialHairType=Blank&clotheType=Hoodie&clotheColor=White&eyeType=Default&eyebrowType=DefaultNatural&mouthType=Default&skinColor=Light"
          ></v-img>
        </v-list-item-avatar>

        <v-list-item-content>
          <v-list-item-title>{{ this.author }}</v-list-item-title>
        </v-list-item-content>

        <v-row align="center" justify="end">
          <!-- <v-btn small plain> comment </v-btn> -->
          <!-- <span class="mr-1">·</span> -->
          <v-icon class="mr-1"> mdi-heart </v-icon>
          <span class="subheading mr-2">{{ this.like }}</span>
          <span class="mr-1">·</span>
          <v-icon class="mr-1"> mdi-share-variant </v-icon>
          <span class="subheading">{{this.share}}</span>
          <v-btn icon @click="show = !show">
            <v-icon>{{ show ? "mdi-chevron-up" : "mdi-chevron-down" }}</v-icon>
          </v-btn>
        </v-row>
      </v-list-item>
    </v-card-actions>
    <v-expand-transition>
      <div v-show="show">
        <v-divider></v-divider>

        <v-text-field
          v-model="newComment"
          label="Add new comment"
          @keydown.enter="create"
          class="px-4"
          color="white"
        >
          <template v-slot:append>
            <v-fade-transition>
              <v-icon v-if="newComment" @click="create"> OK </v-icon>
            </v-fade-transition>
          </template>
        </v-text-field>

        <v-list class="transparent">
          <v-list-item v-for="item in comments" :key="item.cid">
            <v-list-item-avatar color="grey darken-3">
              <v-img
                class="elevation-6"
                alt=""
                src="https://avataaars.io/?avatarStyle=Transparent&topType=ShortHairShortCurly&accessoriesType=Prescription02&hairColor=Black&facialHairType=Blank&clotheType=Hoodie&clotheColor=White&eyeType=Default&eyebrowType=DefaultNatural&mouthType=Default&skinColor=Light"
              ></v-img>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title v-text="item.content"></v-list-item-title>
              <v-list-item-subtitle
                v-text="item.author + ' ' + item.time"
              ></v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </div>
    </v-expand-transition>
  </v-card>
</template>

<script>
export default {
  props: ['pid', 'color'],
  mounted: async function () {
    let post = await this.$axios.$get("/posts/" + this.pid);
    // console.log(post);
    post = post.data;
    this.author = post[0].author;
    this.title = post[0].title;
    this.content = post[0].content;
    this.fruit = post[0].fruit;
    let comments = await this.$axios.$get(
      "/posts/" + this.pid + "/comments"
    );
    // console.log(comments);
    comments = comments.data;
    this.comments = comments.reverse();
    // console.log(this.prop);
    this.like = Math.floor(Math.random() * 100);
    this.share = Math.floor(Math.random() * 50);
  },
  data() {
    return {
      author: "Tom",
      title: "Banana",
      content: "Hello world!",
      fruit: "",
      like: 256,
      show: false,
      comments: [],
      newComment: "",
      share: 45,
    };
  },
  methods: {
    async create() {
      const suc = await this.$axios.$post(
        "/posts/" + this.pid + "/comments",
        {
          author: this.$auth.$storage.getLocalStorage('user'),
          content: this.newComment,
        }
      );
      
      let comments = await this.$axios.$get(
        "/posts/" + this.pid + "/comments"
      );
      // console.log(comments);
      comments = comments.data;
      this.comments = comments.reverse();
      if (suc.result == "success") {
        this.newComment = "";
      }
    },
    async toFruit() {
      this.$router.push('/fruits/' + this.fruit);
    },
  },
};
</script>

<style>
</style>