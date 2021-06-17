<template>
  <div>
    <v-text-field
      v-model="input"
      label="Search your fruit"
      @keydown.enter="search"
      class="px-4"
    >
      <template v-slot:append>
        <v-icon v-if="1" @click="search"> Search </v-icon>
      </template>
    </v-text-field>
    <v-autocomplete
      v-model="marketSelect"
      :items="markets"
      dense
      chips
      label="Market"
      multiple
      solo
    >
      <template v-slot:selection="markets">
        <v-chip
          v-bind="markets.attrs"
          :input-value="markets.selected"
          close
          @click="markets.select"
          @click:close="remove(markets.item)"
        >
          {{ markets.item }}
        </v-chip>
      </template>
    </v-autocomplete>
    <v-row>
      <v-col v-for="(fruit, idx) in this.fruits" :key="idx">
        <now-info v-if="fruit.isHist == 'n'" :name="fruit.name" />
        <fruit-info v-if="fruit.isHist == 'y'" :name="fruit.name" />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import FruitInfo from "../components/FruitInfo.vue";
import NowInfo from "../components/NowInfo.vue";
export default {
  components: { NowInfo, FruitInfo },
  middleware: ["auth"],
  mounted: async function () {
    const m = await this.$axios.$get("/fruits/realtime/markets");
    console.log(m);
    for (let i = 0; i < m.data.length; i++) {
      this.markets.push(m.data[i]["market"]);
    }
    // console.log(this.markets);
    const real = await this.$axios.$post("/fruits/realtime/sub", {
      fruit: "",
      market: this.markets[Math.floor(Math.random() * this.markets.length)],
    });
    // console.log(real);
    if (real.result != "error") {
      let fr = [];
      for (let i = 0; i < 10; i++) {
        fr.push(real.data[i*7%real.data.length]["item"]);
      }
      this.nowFruits = fr;
      for (let i = 0; i < 10; i++) {
        this.fruits.push({ name: this.nowFruits[i], isHist: "n" });
      }
    }
  },
  data: () => ({
    input: "",
    markets: [],
    marketSelect: [],
    fruits: [],
    histFruits: [],
    nowFruits: [],
  }),
  methods: {
    async search() {
      this.nowFruits = [];
      this.histFruits = [];
      this.fruits = [];
      let ff = [];
      for (let i = 0; i < this.marketSelect.length; i++) {
        const real = await this.$axios.$post("/fruits/realtime/sub", {
          fruit: this.input,
          market: this.marketSelect[i],
        });
        if (real.result != "error") {
          console.log(real.data[0]);
          let fr = [];
          for (let i = 0; i < real.data.length; i++) {
            fr.push(real.data[i]["item"]);
          }
          ff = ff.concat(fr);
        }
        // console.log(real);
      }
      // console.log(ff);
      this.nowFruits = ff.filter(this.onlyUnique);
      // console.log(this.fruits);

      const hist = await this.$axios.$post("/fruits/history/sub", {
        fruit: this.input,
      });
      // console.log(Object.keys(hist.data.data_1[0]));
      let h = Object.keys(hist.data.data_1[0]);
      const idx = h.indexOf("時間");
      if (idx >= 0) h.splice(idx, 1);
      this.histFruits = h;
      for (let i = 0; i < h.length; i++) {
        this.fruits.push({ name: h[i], isHist: "y" });
      }
      for (let i = 0; i < this.nowFruits.length; i++) {
        this.fruits.push({ name: this.nowFruits[i], isHist: "n" });
      }
      // console.log(this.histFruits);
    },
    async remove(item) {
      const index = this.marketSelect.indexOf(item);
      if (index >= 0) this.marketSelect.splice(index, 1);
    },
    onlyUnique(value, index, self) {
      return self.indexOf(value) === index;
    },
  },
};
</script>
<style>
</style>