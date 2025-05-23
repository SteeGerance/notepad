<template>


	<img src="https://www.pregerance.fr/wp-content/uploads/2023/08/logo-pregerance-1.png" alt="Logo HubSpot" class="logo"/>
	<h1 class="text-center">
		<a href="/" @click.prevent="goHome" class="brand">🏠 Accueil</a>
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
