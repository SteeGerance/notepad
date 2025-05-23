<template>

  <div class="header" :style="{'background-image': `linear-gradient(to right, cornsilk, cornsilk, ${noteColor})`}">

    <div class="hidden md:block flex-grow text-center items-center mb-0" v-html="banner"></div>

  </div>

    <Editor v-if="state.authKey"></Editor>
    <Home v-else></Home>

</template>

<script>
import Editor from "./Editor.vue";
import Home from "./Home.vue";

import store from "../store";
import {HEADER_TEXT} from "../config";

export default {
  components: {
    Home,
    Editor
  },
  data() {
    return {
      state: store.state,
        banner: HEADER_TEXT
    }
  },
  computed: {
    error() {
      return store.state.error
    },
    noteColor(){
        if (store.state.authKey) {
            const hex = (store.state.authKey || "").substring(0, 8);
            return '#' + hex;
        }

        return 'cornsilk';
    },
    documentIdShort() {

      if (store.state.authKey) {
        return (store.state.authKey || "").substring(0, 8);
      }

      return null;
    }
  },
  methods: {
    goHome() {
      store.actions.reset();
    }
  }
}
</script>
