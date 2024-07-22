<template>
  <div>
    <transition name="fade" mode="out-in">
      <div v-if="!isLandscape" key="message" class="landscape-overlay" >
        <div class="rotate-device-message">请旋转设备以横屏模式查看</div>
      </div>
      <div v-else key="content">
        <slot></slot>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isLandscape: false,
    };
  },
  mounted() {
    this.checkOrientation();
    window.addEventListener("resize", this.checkOrientation);
    window.addEventListener("orientationchange", this.checkOrientation);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.checkOrientation);
    window.removeEventListener("orientationchange", this.checkOrientation);
  },
  methods: {
    checkOrientation() {
      if (window.screen.orientation) {
        this.isLandscape =
          window.screen.orientation.type.startsWith("landscape");
      } else if (window.orientation !== undefined) {
        this.isLandscape = Math.abs(window.orientation) === 90;
      } else {
        this.isLandscape = window.innerWidth > window.innerHeight;
      }

      this.forceOrientation();
    },
    forceOrientation() {
      const rootElement = document.documentElement;

      if (this.isLandscape) {
        rootElement.style.transform = "none";
        rootElement.style.width = "100%";
        rootElement.style.height = "100%";
        rootElement.style.position = "static";
      } else {
        rootElement.style.transform = "rotate(-90deg)";
        rootElement.style.transformOrigin = "left top";
        rootElement.style.width = "100vh";
        rootElement.style.height = "100vw";
        rootElement.style.position = "absolute";
        rootElement.style.top = "0";
        rootElement.style.left = "0";
      }
    },
  },
};
</script>

<style scoped>
.landscape-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 20px;
  box-sizing: border-box;
}

.rotate-device-message {
  color: #fff;
  font-size: 24px;
  font-weight: bold;
  border: 2px solid #fff;
  padding: 20px;
  border-radius: 10px;
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
