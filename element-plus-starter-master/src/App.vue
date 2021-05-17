<template>
  <div>
    <div class="flex">
      <div v-show="date">{{ date }}</div>

      <div v-show="time">{{ time }}</div>
    </div>
    <img class="logo" alt="Vue logo" src="" />
    <HelloWorld :msg="msg" />
    <div class="btn">
      <el-button type="primary" @click="startHacking"> Get Started </el-button>
    </div>
    <comment-element class="w-8/12 mx-auto" />
  </div>
</template>

<script>
import { ref } from "vue";
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "CurrentDate",
  data: () => ({
    timestamp: "",
    date: "",
    time: "",
    currentYear: "",
  }),
  props: {
    msg: String,
  },
  methods: {
    getDate: function() {
      return new Date().toLocaleDateString();
    },
    getTime: function() {
      return new Date().toLocaleTimeString();
    },
  },
  mounted: function() {
    this.date = this.getDate();
    this.time = this.getTime();
  },
  name: "App",
  components: {
    HelloWorld,
  },

  setup() {
    const msg = ref(
      "Welcome to Element Plus, a Vue 3.0 based component library"
    );
    const startHacking = () => {
      msg.value = "Start coding with Element Plus with 💖";
    };
    return {
      msg,
      startHacking,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;

  margin-top: 60px;
}
.logo {
  width: 50%;
}
.btn {
  color: #2c3e50;
  margin-top: 100px;
}
</style>
