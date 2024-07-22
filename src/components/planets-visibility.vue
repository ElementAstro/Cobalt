<template>
  <v-dialog max-width="600" v-model="$store.state.showPlanetsVisibilityDialog">
    <v-card v-if="$store.state.showPlanetsVisibilityDialog" class="dark-card">
      <v-card-title>
        <div class="text-h5 white--text">{{ $t("Planets Visibility") }}</div>
      </v-card-title>
      <v-card-text class="white--text">
        {{
          $t("Night from {0} to {1}", [
            startDate.format("MMMM Do"),
            endDate.format("MMMM Do"),
          ])
        }}
      </v-card-text>
      <v-card-text>
        <div class="white--text">
          <v-row no-gutters>
            <v-col cols="1" offset="2"
              ><span>{{ $t("Rise") }}</span></v-col
            >
            <v-col cols="1"
              ><span>{{ $t("Set") }}</span></v-col
            >
            <v-col cols="8">
              <v-row justify="space-between">
                <span>12:00</span><span>18:00</span><span>00:00</span
                ><span>06:00</span><span>12:00</span>
              </v-row>
            </v-col>
          </v-row>
          <template v-for="obj in objs">
            <v-row no-gutters :key="obj.v">
              <v-col cols="2">{{ cleanName(obj) }}</v-col>
              <v-col cols="1">{{
                formatTime(obj.computeVisibility()[0].rise)
              }}</v-col>
              <v-col cols="1">{{
                formatTime(obj.computeVisibility()[0].set)
              }}</v-col>
              <v-col cols="8">
                <div :style="sunBackgroundStr">
                  &nbsp;
                  <div v-html="planetBackgroundStr(obj)"></div>
                </div>
              </v-col>
            </v-row>
          </template>
        </div>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          class="blue--text darken-1"
          text
          @click.native="$store.state.showPlanetsVisibilityDialog = false"
          >Close</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import Moment from "moment";
import swh from "@/assets/sw_helpers.js";

export default {
  data() {
    return {
      objs: [
        this.$stel.getObj("NAME Sun"),
        this.$stel.getObj("NAME Moon"),
        this.$stel.getObj("NAME Mercury"),
        this.$stel.getObj("NAME Venus"),
        this.$stel.getObj("NAME Mars"),
        this.$stel.getObj("NAME Jupiter"),
        this.$stel.getObj("NAME Saturn"),
      ],
    };
  },
  methods: {
    formatTime(jdm) {
      const d = new Date();
      d.setMJD(jdm);
      const utc = Moment.utc(d);
      utc.local();
      return utc.format("HH:mm");
    },
    cleanName(obj) {
      return swh.cleanupOneSkySourceName(obj.designations()[0]);
    },
    planetBackgroundStr(obj) {
      const d = new Date();
      d.setMJD(obj.computeVisibility()[0].rise);
      const rise = Moment.utc(d);
      rise.local();
      d.setMJD(obj.computeVisibility()[0].set);
      const set = Moment.utc(d);
      set.local();

      const hourToPercent = (h) =>
        h >= 12
          ? Math.round(((h - 12) / 24) * 100)
          : Math.round(((h + 12) / 24) * 100);

      const riseP = hourToPercent(rise.hours());
      const setP = hourToPercent(set.hours());
      if (setP > riseP) {
        return `<div style='z-index: 100; position: absolute; background-color: rgb(200, 200, 50); left: ${riseP}%; min-width: ${
          setP - riseP
        }%; top: 7px; height: 8px;'></div>`;
      } else {
        let ret = `<div style='z-index: 100; position: absolute; background-color: rgb(200, 200, 50); left: 0%; min-width: ${setP}%; top: 7px; height: 8px;'></div>`;
        ret += `<div style='z-index: 100; position: absolute; background-color: rgb(200, 200, 50); right: 0%; min-width: ${
          100 - riseP
        }%; top: 7px; height: 8px;'></div>`;
        return ret;
      }
    },
  },
  computed: {
    sunBackgroundStr() {
      var sun = this.$stel.getObj("NAME Sun");
      const brightness = [];
      const d = new Moment(this.startDate);

      const obs = this.$stel.core.observer.clone();
      for (let i = 0; i < 25; i++) {
        obs.utc = d.toDate().getMJD();
        d.local();
        const azalt = this.$stel.convertFrame(
          obs,
          "ICRF",
          "OBSERVED",
          sun.getInfo("radec", obs)
        );
        const alt =
          (this.$stel.anpm(this.$stel.c2s(azalt)[1]) * 180.0) / Math.PI;
        brightness.push(alt / (Math.PI / 2));
        d.add(1, "hours");
      }
      obs.destroy();

      let txt = "position: relative; background: linear-gradient(to right, ";
      for (let i = 0; i < 25; i++) {
        let bi = (brightness[i] + 0.1) * 5;
        bi = bi > 1 ? 1 : bi;
        bi = bi < 0 ? 0 : bi;
        txt += `rgb(${Math.round(53 * bi)}, ${Math.round(
          173 * bi
        )}, ${Math.round(211 * bi)}) ${Math.round((i / 24) * 100)}% `;
        if (i !== 24) {
          txt += ",";
        }
      }
      txt += "); min-width: 100%; height: 100%";
      return txt;
    },
    startDate() {
      var sun = this.$stel.getObj("NAME Sun");
      let u = this.$store.state.stel.observer.utc;
      if (u < sun.rise) {
        // It's still night (in the morning), display last night's planets visibility
        u = u - 1;
      }
      let d = new Date();
      d.setMJD(u);
      d = new Moment(d);
      d.local();
      d.hours(12);
      d.minutes(0);
      d.seconds(0);
      return d;
    },
    endDate() {
      const d = new Moment(this.startDate);
      d.add(1, "d");
      return d;
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap");

body {
  font-family: "Roboto", sans-serif;
}

.dark-card {
  background-color: #1e1e1e !important;
  color: #e0e0e0 !important;
}

.text-h5 {
  color: #e0e0e0 !important;
}

.v-card-title {
  border-bottom: 1px solid #333;
}

.v-card-text {
  color: #e0e0e0;
}

.v-btn {
  color: #007bff !important;
}

.modal {
  background: rgba(0, 0, 0, 0.7);
}

.close-button {
  background: transparent;
  border: none;
  color: #e0e0e0;
  font-size: 24px;
  cursor: pointer;
  transition: color 0.3s;
}

.close-button:hover {
  color: #ff5c5c;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
