<template>
  <v-card
    :v-show="exist"
    class="mx-auto text-center my-5"
    color="#303030"
    dark
    max-width="600"
  >
    <v-card-title class="text-h4 font-weight-black">
      Last Years' Statistics
    </v-card-title>
    <v-sparkline
      :value="value"
      :gradient="gradient"
      :smooth="radius || false"
      :padding="padding"
      :line-width="width"
      :stroke-linecap="lineCap"
      :gradient-direction="gradientDirection"
      :fill="fill"
      :type="type"
      :auto-line-width="autoLineWidth"
      auto-draw
    ></v-sparkline>
    <v-divider></v-divider>

    <v-list class="transparent">
      <v-list-item class="my-5">
        <v-row align="start">
          <div class="text-caption grey--text text-uppercase mx-5">
            avg origin price
          </div>
          <div>
            <span class="text-h3 font-weight-black" v-text="avg || '—'"></span>
            <strong v-if="avg">$/year</strong>
          </div>
        </v-row>
      </v-list-item>

      <v-list-item class="my-5">
        <v-row align="start">
          <div class="text-caption grey--text text-uppercase mx-5">
            產季 from
          </div>
          <div>
            <span
              class="text-h4 font-weight-black"
              v-text="startDate || '—'"
            ></span>
          </div>
          <div class="text-caption grey--text text-uppercase mx-5">to</div>
          <div>
            <span
              class="text-h4 font-weight-black"
              v-text="endDate || '—'"
            ></span>
          </div>
        </v-row>
      </v-list-item>
    </v-list>
  </v-card>
</template>

<script>
const gradients = [
  ["#222"],
  ["#42b3f4"],
  ["red", "orange", "yellow"],
  ["purple", "violet"],
  ["#00c6ff", "#F0F", "#FF0"],
  ["#f72047", "#ffd200", "#1feaea"],
];

export default {
  props: {
    name: String,
  },
  mounted: async function () {
    const v = await this.$axios.$post("/fruits/history", {
      fruit: this.name,
    });
    if (v.result != "error") {
      // console.log(v);
      this.currentPrice =
        v.data.data_1[v.data.data_1.length - 1][this.name].toFixed(1);
      let vs = [];
      for (let i = 0; i < v.data.data_season.length; i++) {
        vs.push(v.data.data_season[i][this.name]);
      }
      this.value = vs;

      const season = await this.$axios.$post("/fruits/history/season", {
        fruit: this.name,
      });
      let start = new Date(season.data[0]["season_start"]);
      let end = new Date(season.data[0]["season_end"]);
      // console.log(start);
      // console.log(end);
      this.startDate =
        start.getFullYear() +
        "-" +
        (start.getMonth() + 1) +
        "-" +
        start.getDate();
      this.endDate =
        end.getFullYear() + "-" + (end.getMonth() + 1) + "-" + end.getDate();
    }
  },
  data: () => ({
    width: 1,
    radius: 2,
    padding: 8,
    lineCap: "round",
    gradient: gradients[5],
    value: [0, 2, 5, 9, 5, 10, 3, 5, 0, 0, 1, 8, 2, 9, 0],
    gradientDirection: "top",
    gradients,
    fill: true,
    type: "trend",
    autoLineWidth: false,
    startDate: "",
    endDate: "",
    exist: false,
  }),
  computed: {
    avg() {
      const sum = this.value.reduce((acc, cur) => acc + cur, 0);
      const length = this.value.length;

      if (!sum && !length) return 0;

      return Math.ceil(sum / length);
    },
  },
};
</script>
