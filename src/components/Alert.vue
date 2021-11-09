<template>
  <div v-if="visible" :class="[userPosition, userColor]" class="w-full lg:w-3/12 border-l-4 p-4" role="alert">
    <p class="font-bold">{{ title }}</p>
    <p>{{ description }}</p>
  </div>
</template>

<script>
import "animate.css";
export default {
  name: "Alert",
  props: {
    visible: Boolean,
    position: String,
    title: String,
    description: String,
    color: String
  },
  data() {
    return {
      userPosition: "",
      userColor: ""
    };
  },
  watch: {
    visible() {
      switch (this.color) {
        case "success":
          this.userColor = this.success;
          break;
        case "error":
          this.userColor = this.error;
          break;
        case "warnning":
          this.userColor = this.warnning;
          break;
        default:
          this.userColor = this.success;
      }
      switch (this.position) {
        case "top-right":
          this.userPosition = this.topRight;
          break;
        case "bottom-right":
          this.userPosition = this.bottomRight;
          break;
        case "top-left":
          this.userPosition = this.topLeft;
          break;
        case "bottom-left":
          this.userPosition = this.bottomLeft;
          break;
        default:
          this.userPosition = this.topRight;
      }
    }
  },
  computed: {
    topRight() {
      return "top-0 right-0 mt-5 mr-5 lg:absolute animate__animated animate__fadeInRight";
    },
    topLeft() {
      return "top-0 left-0 mt-5 ml-5 lg:absolute animate__animated animate__fadeInLeft";
    },
    bottomRight() {
      return "bottom-0 right-0 mb-5 mr-5 lg:absolute animate__animated animate__fadeInRight";
    },
    bottomLeft() {
      return "bottom-0 left-0 mb-5 ml-5 lg:absolute animate__animated animate__fadeInLeft";
    },
    success() {
      return "bg-green-100 border-green-500 text-green-700";
    },
    error() {
      return "bg-red-100 border-red-500 text-red-700";
    },
    warnning() {
      return "bg-yellow-100 border-yellow-500 text-yellow-700";
    }
  }
};
</script>

