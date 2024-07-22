<!-- Modal.vue -->
<template>
  <transition name="fade">
    <div class="modal" v-if="visible" @click.self="close">
      <div class="modal-content">
        <button class="close-button" @click="close">&times;</button>
        <slot></slot>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    visible: {
      type: Boolean,
      required: true,
    },
  },
  methods: {
    close() {
        this.$store.state.showScanner = !this.$store.state.showScanner;
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap");

body {
  font-family: "Roboto", sans-serif;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: #2e2e2e;
  color: #e0e0e0;
  padding: 20px;
  border-radius: 8px;
  position: relative;
  max-width: 600px;
  width: 100%;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
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
