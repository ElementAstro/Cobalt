<template>
  <div class="btn-bar-container" style="position: absolute; bottom: 0; width: 100%; display: flex; justify-content: center;">
    <v-slide-x-transition>
      <div v-if="showButtons" class="btn-bar" style="display: flex; align-items: center;">
        <v-spacer></v-spacer>

        <bottom-button
          :label="$t('Constellations')"
          v-if="$store.state.showConstellationsLinesButton !== false"
          :img="require('@/assets/images/btn-cst-lines.svg')"
          img_alt="Constellations Button"
          :toggled="$store.state.stel.constellations.lines_visible"
          @clicked="
            (b) => {
              $stel.core.constellations.lines_visible = b;
              $stel.core.constellations.labels_visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Constellations Art')"
          v-if="$store.state.showConstellationsArtButton !== false"
          :img="require('@/assets/images/btn-cst-art.svg')"
          img_alt="Constellations Art Button"
          :toggled="$store.state.stel.constellations.images_visible"
          @clicked="
            (b) => {
              $stel.core.constellations.images_visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Atmosphere')"
          v-if="$store.state.showAtmosphereButton !== false"
          :img="require('@/assets/images/btn-atmosphere.svg')"
          img_alt="Atmosphere Button"
          :toggled="$store.state.stel.atmosphere.visible"
          @clicked="
            (b) => {
              $stel.core.atmosphere.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Landscape')"
          v-if="$store.state.showLandscapeButton !== false"
          :img="require('@/assets/images/btn-landscape.svg')"
          img_alt="Landscape Button"
          :toggled="$store.state.stel.landscapes.visible"
          @clicked="
            (b) => {
              $stel.core.landscapes.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Azimuthal Grid')"
          v-if="$store.state.showAzimuthalGridButton !== false"
          :img="require('@/assets/images/btn-azimuthal-grid.svg')"
          img_alt="Azimuthal Button"
          :toggled="$store.state.stel.lines.azimuthal.visible"
          @clicked="
            (b) => {
              $stel.core.lines.azimuthal.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Equatorial Grid')"
          v-if="$store.state.showEquatorialGridButton !== false"
          :img="require('@/assets/images/btn-equatorial-grid.svg')"
          img_alt="Equatorial Grid Button"
          :toggled="$store.state.stel.lines.equatorial_jnow.visible"
          @clicked="
            (b) => {
              $stel.core.lines.equatorial_jnow.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Equatorial J2000 Grid')"
          v-if="$store.state.showEquatorialJ2000GridButton !== false"
          :img="require('@/assets/images/btn-equatorial-grid.svg')"
          img_alt="Equatorial J2000 Grid Button"
          :toggled="$store.state.stel.lines.equatorial.visible"
          @clicked="
            (b) => {
              $stel.core.lines.equatorial.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Deep Sky Objects')"
          :img="require('@/assets/images/btn-nebulae.svg')"
          img_alt="Deep Sky Objects Button"
          :toggled="$store.state.stel.dsos.visible"
          @clicked="
            (b) => {
              $stel.core.dsos.visible = b;
            }
          "
        >
        </bottom-button>
        <bottom-button
          :label="$t('Night Mode')"
          v-if="$store.state.showNightmodeButton !== false"
          :img="require('@/assets/images/btn-night-mode.svg')"
          img_alt="Night Mode Button"
          :toggled="$store.state.nightmode"
          @clicked="
            (b) => {
              setNightMode(b);
            }
          "
        >
        </bottom-button>
        <bottom-button :label="$t('Fullscreen')"
                    :img="fullscreenBtnImage"
                    img_alt="Fullscreen Button"
                    class="mr-auto hidden-xs-only"
                    :toggled="$store.state.fullscreen"
                    @clicked="(b) => { setFullscreen(b) }">
        </bottom-button>

        <v-btn icon @click="toggleButtons" class="ml-2">
          <v-icon>mdi-chevron-down</v-icon>
        </v-btn>

        <v-spacer></v-spacer>
      </div>
    </v-slide-x-transition>
    <v-slide-x-transition>
      <v-btn v-if="!showButtons" icon @click="toggleButtons" class="expand-btn">
        <v-icon>mdi-chevron-up</v-icon>
      </v-btn>
    </v-slide-x-transition>
  </div>
</template>

<script>
import BottomButton from "@/components/bottom-button.vue";

export default {
  components: { BottomButton },
  data: function () {
    return {
      showButtons: true,
    };
  },
  computed: {
    fullscreenBtnImage: function () {
      return this.$store.state.fullscreen
        ? require("@/assets/images/svg/ui/fullscreen_exit.svg")
        : require("@/assets/images/svg/ui/fullscreen.svg");
    }
  },
  methods: {
    setFullscreen: function (b) {
      this.$fullscreen.toggle(document.body, {
        wrap: false,
        callback: this.onFullscreenChange,
      });
    },
    setNightMode: function (b) {
      this.$store.commit("toggleBool", "nightmode");
      if (window.navigator.userAgent.indexOf("Edge") > -1) {
        document.getElementById("nightmode").style.opacity = b ? "0.5" : "0";
      }
      document.getElementById("nightmode").style.visibility = b
        ? "visible"
        : "hidden";
    },
    onFullscreenChange: function (b) {
      if (this.$store.state.fullscreen === b) return;
      this.$store.commit("toggleBool", "fullscreen");
    },
    toggleButtons() {
      this.showButtons = !this.showButtons;
    },
  },
};
</script>

<style scoped>
.btn-bar {
  border-radius: 10px;
  backdrop-filter: blur(5px);
  background-color: rgba(0, 0, 0, 0.1);
  padding: 0.5rem;
}

.expand-btn {
  border: 1px solid rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  backdrop-filter: blur(5px);
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  margin-left: 8px;
}
</style>
