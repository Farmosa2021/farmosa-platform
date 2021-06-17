<template>
  <v-card
    v-if="this.d.length > 0"
    class="mx-auto text-center my-5"
    color="#303030"
    dark
    max-width="600"
  >
    <!-- <v-card-title class="text-h4 font-weight-black">
      Last Years' Statistics
    </v-card-title> -->
    <v-window v-model="onboarding" reverse>
      <v-window-item v-for="(n, idx) in d" :key="idx">
        <v-list class="transparent">
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                avg price
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="'$' + n.avg_price || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                high price
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="'$' + n.high_price || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                mid price
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="'$' + n.mid_price || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                low price
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="'$' + n.low_price || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                market
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="n.market || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
          <v-list-item class="my-5">
            <v-row align="start">
              <div class="text-caption grey--text text-uppercase mx-5">
                amount
              </div>
              <div>
                <span
                  class="text-h3 font-weight-black"
                  v-text="n.amount || '—'"
                ></span>
                <!-- <strong>$/year</strong> -->
              </div>
            </v-row>
          </v-list-item>
        </v-list>
      </v-window-item>
    </v-window>

    <v-card-actions class="justify-space-between">
      <v-btn text @click="prev">
        <v-icon>mdi-chevron-left</v-icon>
      </v-btn>
      <v-item-group v-model="onboarding" class="text-center" mandatory>
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
    for (let i = 0; i < mar.length; i++) {
      const real = await this.$axios.$post("/fruits/realtime", {
        fruit: this.name,
        market: mar[i],
      });
      //   console.log(real);
      if (real.result != "no result") {
        let item = {};
        // console.log(real.data[0]);
        item.low_price = real.data[0].low_price.toFixed(1);
        item.mid_price = real.data[0].mid_price.toFixed(1);
        item.high_price = real.data[0].high_price.toFixed(1);
        item.avg_price = real.data[0].avg_price.toFixed(1);
        item.market = real.data[0].market;
        item.date = real.data[0].date;
        item.amount = real.data[0].amount;
        // console.log(this.d);
        this.d.push(item);
      }
    }
    this.length = this.d.length;
    // console.log(this.d);
  },
  data: () => ({
    d: [],
    length: 3,
    onboarding: 0,
  }),

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

<style>
</style>