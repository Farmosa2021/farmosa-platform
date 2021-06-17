<template>
  <v-card v-if="this.show>1" class="mx-auto text-center" color="green" dark max-width="600">
    <v-card-text >
      <v-window v-model="onboarding1" reverse>
        <v-window-item v-for="(v, idx) in value" :key="idx">
          <v-sheet class="pa-3" color="rgba(0, 0, 0, .12)">
          <v-sparkline
            :value="value[idx]"
            color="rgba(255, 255, 255, .7)"
            height="100"
            padding="24"
            stroke-linecap="round"
            type="bar"
            :labels="label[idx]"
          >
            <!-- <template v-slot:label="item"> {{ item.value }} </template> -->
          </v-sparkline>
          </v-sheet>
        </v-window-item>
      </v-window>
    </v-card-text>

    <v-card-text>
      <div class="text-h4 font-weight-thin">Sales Amount</div>
    </v-card-text>

    <v-divider></v-divider>
    <v-card-actions class="justify-space-between">
      <v-btn text @click="prev">
        <v-icon>mdi-chevron-left</v-icon>
      </v-btn>
      <v-item-group v-model="onboarding1" class="text-center" mandatory>
        <v-item
          v-for="n in length"
          :key="`btn-${n}`"
          v-slot="{ active, toggle }"
        >
          <v-btn :input-value="active" icon @click="toggle">
            <v-icon>mdi-record</v-icon>
          </v-btn>
        </v-item>
      </v-item-group>
      <v-btn text @click="next">
        <v-icon>mdi-chevron-right</v-icon>
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  props: ["name"],
  mounted: async function () {
    const m = await this.$axios.$get("/fruits/realtime/markets");
    let mar = [];
    for (let i = 0; i < m.data.length; i++) {
      mar.push(m.data[i]["market"]);
    }
    // console.log(mar);
    let val = [];
    let mark = [];
    let lab = [];
    for (let i = 0; i < mar.length; i++) {
      const real = await this.$axios.$post("/fruits/realtime", {
        fruit: this.name,
        market: mar[i],
      });
      //   console.log(real);
      if (real.result != "no result") {
        val.push(real.data[0].amount);
        mark.push(real.data[0].market);
        lab.push(real.data[0].market + " : " + real.data[0].amount);
      }
      if (val.length >= 5) {
        this.value.push(val);
        this.market.push(mark);
        this.label.push(lab);
        val = [];
        mark = [];
        lab = [];
      }
    }
    if (val.length > 0) {
      this.value.push(val);
      this.market.push(mark);
      this.label.push(lab);
    }
    this.length = this.value.length;
    console.log(this.value[0].length);
    this.show = this.value[0].length;
  },
  data: () => ({
    market: [],
    value: [],
    label: [],
    length: 1,
    onboarding1: 0,
    show: 0,
  }),
  methods: {
    next() {
      this.onboarding1 =
        this.onboarding1 + 1 === this.length ? 0 : this.onboarding1 + 1;
    },
    prev() {
      this.onboarding1 =
        this.onboarding1 - 1 < 0 ? this.length - 1 : this.onboarding1 - 1;
    },
  },
};
</script>
