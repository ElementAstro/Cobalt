<template>
  <div
    class="bottom-button"
    :class="{ on: toggled }"
    @click="handleClick"
    tabindex="0"
    @blur="handleBlur"
  >
    <a>
      <img :src="img" :alt="img_alt" />
    </a>
    <transition name="fade">
      <div v-if="showHint" class="hint">{{ label }}</div>
    </transition>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap");

.bottom-button {
  width: 40px;
  height: 40px;
  position: relative;
  display: inline-block;
  user-select: none;
  backdrop-filter: blur(5px);
  background-color: rgba(30, 30, 30, 0.8);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  bottom: 20px;
  margin-right: 15px;
  transition: background-color 0.3s, transform 0.3s;
  outline: none;
}

.bottom-button:hover,
.bottom-button:focus {
  background-color: rgba(50, 50, 50, 0.8);
  transform: translateY(-5px);
}

.bottom-button img {
  width: 100%;
  height: 100%;
  filter: opacity(0.7);
  transition: filter 0.3s;
}

.bottom-button.on img {
  filter: opacity(1);
}

.bottom-button .hint {
  opacity: 0;
  visibility: hidden;
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%) translateY(10px);
  color: white;
  background: rgba(0, 0, 0, 0.7);
  padding: 5px 10px;
  border-radius: 5px;
  width: max-content;
  max-width: 200px;
  font-size: 12px;
  text-align: center;
  white-space: nowrap;
  z-index: 1;
  transition: opacity 0.3s, transform 0.3s, visibility 0.3s;
}

.bottom-button .hint-visible {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) translateY(0);
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

<script>
export default {
  name: "bottom-button",
  props: ["label", "img", "toggled", "img_alt"],
  data() {
    return {
      showHint: false,
    };
  },
  methods: {
    handleClick() {
      const b = !this.toggled;
      this.$emit("clicked", b);
      this.showHint = true;

      if (!this.isMobile()) {
        setTimeout(() => {
          this.showHint = false;
        }, 1000);
      }
    },
    handleBlur() {
      if (!this.isMobile()) {
        this.showHint = false;
      }
    },
    isMobile() {
      return /Mobi|Android/i.test(navigator.userAgent);
    },
  },
};
</script>
