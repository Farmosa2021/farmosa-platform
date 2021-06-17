
<template>
  <v-card class="mx-auto my-5 text-center" color="green" dark max-width="600">
    <v-list-item>
      <v-card-title class="text-h4 font-weight-black">
        Origin price
      </v-card-title>
      <!-- <v-row align="center" justify="end" class="mx-1">
        <v-btn icon dark @click="addFav">
          <v-icon>
            {{ active ? "mdi-heart" : "mdi-heart-outline" }}
          </v-icon>
        </v-btn>
      </v-row> -->
    </v-list-item>
    <v-card-text>
      <v-window v-model="onboarding" vertical>
        <v-window-item v-for="(v, index) in this.value" :key="index">
          <v-sparkline
            :value="v"
            color="rgba(255, 255, 255, .7)"
            height="100"
            padding="24"
            stroke-linecap="round"
            smooth
            line-width="1.2"
          >
          </v-sparkline>
        </v-window-item>
      </v-window>
    </v-card-text>
    <v-divider></v-divider>
    <v-item-group
      active-class="deep-purple accent-4 white--text"
      column
      v-model="onboarding"
    >
      <v-item v-for="n in length" :key="n" v-slot="{ active, toggle }">
        <v-btn class="mx-4 my-2" rounded :input-value="active" @click="toggle">
          {{ text[n - 1] }}
        </v-btn>
      </v-item>
    </v-item-group>
    <!-- <v-card-actions class="justify-center">
      <v-btn
        block
        text
      >
        Go to Report
      </v-btn>
    </v-card-actions> -->
    <v-list class="transparent">
      <v-list-item class="my-5">
        <div class="text-caption text-uppercase mx-5">current Price</div>
        <div>
          <span
            class="text-h3 font-weight-black"
            v-text="'$' + currentPrice || '—'"
          ></span>
          <strong>/斤</strong>
        </div>
      </v-list-item>

      <v-window v-model="onboarding" vertical>
        <v-window-item v-for="(v, index) in this.value" :key="index">
          <v-list-item class="my-5">
            <div class="text-caption text-uppercase mx-5">Max Price</div>
            <div>
              <span
                class="text-h3 font-weight-black"
                v-text="'$' + Math.max.apply(null, v).toFixed(1) || '—'"
              ></span>
              <strong>/斤</strong>
            </div>
          </v-list-item>
        </v-window-item>
      </v-window>

      <v-window v-model="onboarding" vertical>
        <v-window-item v-for="(v, index) in this.value" :key="index">
          <v-list-item class="my-5">
            <div class="text-caption text-uppercase mx-5">Min Price</div>
            <div>
              <span
                class="text-h3 font-weight-black"
                v-text="'$' + Math.min.apply(null, v).toFixed(1) || '—'"
              ></span>
              <strong>/斤</strong>
            </div>
          </v-list-item>
        </v-window-item>
      </v-window>
    </v-list>
  </v-card>
</template>


<script>
export default {
  props: {
    name: String,
  },
  data: () => ({
    value: [
      // [423, 446, 675, 510, 590, 610, 760],
      // [1, 2, 3, 4, 5, 4, 3],
      // [5, 5, 5, 5, 5, 5, 5],
    ],
    length: 3,
    onboarding: 0,
    text: ["1 year", "6 month", "3 month"],
    currentPrice: 0,
    active: false,
    exist: false,
  }),
  mounted: async function () {
    const v = await this.$axios.$post("/fruits/history", {
      fruit: this.name,
    });
    if (v.result != "error") {
      this.exist = true;
      // console.log(v);
      this.currentPrice =
        v.data.data_1[v.data.data_1.length - 1][this.name].toFixed(1);
      // this.value.push(v.data.data_6);
      // this.value.push(v.data.data_3);
      // this.value.push(v.data.data_1);
      let v6 = [];
      for (let i = 0; i < v.data.data_6.length; i++) {
        v6.push(v.data.data_6[i][this.name]);
      }
      this.value.push(v6);
      let v3 = [];
      for (let i = 0; i < v.data.data_3.length; i++) {
        v3.push(v.data.data_3[i][this.name]);
      }
      this.value.push(v3);
      let v1 = [];
      for (let i = 0; i < v.data.data_1.length; i++) {
        v1.push(v.data.data_1[i][this.name]);
      }
      this.value.push(v1);
      // console.log(fav.data);
    }
  },
  methods: {
    next() {
      this.onboarding =
        this.onboarding + 1 === this.length ? 0 : this.onboarding + 1;
    },
    prev() {
      this.onboarding =
        this.onboarding - 1 < 0 ? this.length - 1 : this.onboarding - 1;
    },

  },
};
</script>