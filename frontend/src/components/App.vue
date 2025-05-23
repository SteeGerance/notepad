<template>
	<style>
	a:link, a:visited {
	  color: grey;
	  text-align: center;
	  text-decoration: none;
	  display: inline-block;
	}

    a:hover, a:active {
      color: black;
    }
    
    h1 {
    	text-align:center;
       }

	</style>


	<h1 class="mb-0">
		<span class="hidden md:inline mr-2 text-blue-400">🏠</span>
		<a href="/" @click.prevent="goHome" class="brand">Accueil</a>
		<template v-if="documentIdShort">
			<span class="mx-2">&ndash;</span>#{{ documentIdShort }}
		</template>
	</h1>
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
