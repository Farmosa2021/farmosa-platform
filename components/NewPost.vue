<template>
  <v-row justify="center">
    <v-dialog v-model="dialog" persistent max-width="600px">
      <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on" plain> NEW POST </v-btn>
      </template>
      <v-card>
        <v-alert type="error" :value="error" transition="fade-transition"> Error. </v-alert>
        <v-alert type="success" :value="success" transition="fade-transition"> Success. </v-alert>
        <v-card-title>
          <span class="text-h5">New Post</span>
        </v-card-title>
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
          <v-btn color="blue darken-1" text @click="dialog = false">
            Close
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="newPost">
            Post
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  props: {
    author: String,
  },
  data: () => ({
    dialog: false,
    fruit: [''],
    // items: ['Trevor Handsen', 'Alex Nelson'],
    title: '',
    content: '',
    success: false,
    error: false,
  }),
  methods: {
    async newPost() {
      let suc = await this.$axios.$post("/posts/", {
        author: this.author,
        title: this.title,
        fruit: this.fruit[0],
        content: this.content,
      });
      console.log(suc);
      if(suc.response.result=='success'){
        this.success = true;
        await this.sleep(1000);
        this.success = false;
        // this.$router.push('/users/' + this.author);
        this.dialog = false;
      }else{
        this.error = true;
        await this.sleep(1000);
        this.error = false;
        this.dialog = false;
      }
    },
    async sleep(ms = 0) {
      return new Promise(r => setTimeout(r, ms));
    }
  }
};
</script>
