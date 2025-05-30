<template>
  <div class="w-full max-w-screen-lg mx-auto flex flex-col flex-grow mt-5 px-1 lg:px-0">


    <div id="plain_text_not_ace" class="flex flex-col flex-grow">

      <textarea @keyup="textKeyUp" @keydown="textKeyDown" @select="textSelect" @mouseup="textSelect" v-model="text"
                placeholder="Ecrivez ici." style="resize: none"
                class="flex-grow"></textarea>

      <div class="flex justify-between mt-2 mb-5">

        <div class="status-bar">
          Mots: {{ wordCount }},
          Caractères: <span v-if="selectedCharCount">{{ selectedCharCount }}/</span>{{ charCount }},
          Lignes: {{ lineCount }}
        </div>

        <div>
          <a href="" @click.prevent="deleteForever" class="font-bold text-red-500">Détruire totalement</a>
        </div>

      </div>

    </div>

  </div>
</template>

<script>
import {watch} from "vue";

import {TextUtil} from "../classes/TextUtil";
import debounce from "debounce";
import store from "../store";
import {SecureStorage} from "../classes/SecureStorage";

// https://github.com/reddit-archive/reddit/blob/753b17407e9a9dca09558526805922de24133d53/r2/r2/models/link.py#L149
const TEXT_MAX_LEN = 40000;

export default {
  data() {
    return {
      isBusy: true,
      text: '',
      // stats
      charCount: 0,
      selectedCharCount: 0,
      wordCount: 0,
      lineCount: 0
    }
  },
  watch: {
    // nothing
  },
  methods: {
    writeNow: async function () {

      try {

        const text = this.text;
        const authKey = store.state.authKey;
        const password = store.state.encryptionKey;

        await SecureStorage.write(authKey, text, password);

      } catch (ex) {
        const errorString = ex?.message || ex;
        alert('Something went wrong during saving: ' + errorString);
      }

    },
    writeLater: debounce(async function () {
      await this.writeNow();
    }, 800),
    textKeyDown(event) {

      if (event.ctrlKey || event.metaKey) {
        switch (String.fromCharCode(event.which).toLowerCase()) {
          case 's':
            event.preventDefault();
            break;
        }
      }
    },
    async textKeyUp() {
      this.writeLater();
    },
    textSelect(event) {

      const target = event.target;

      let val = target.value;
      let start = target.selectionStart;
      let end = target.selectionEnd;

      if (start === end) {
        this.selectedCharCount = 0;
      } else {

        let selected = val.substring(start, end);
        this.selectedCharCount = selected.length;
      }

    },
    deleteForever() {

      if (confirm('Confirmer la suppression ?')) {

            const key = store.state.authKey;

            SecureStorage.remove(key).then(() => {
                store.actions.reset();
            }).catch(() => {
                alert('Failed to delete');
            });
      }

    }
  },
  mounted() {

    // Logged in as new user!
    watch(() => store.state.authKey, async (newValue, oldValue) => {

      this.isBusy = true;

      if (newValue) {
          const password = store.state.encryptionKey;
          const text = await SecureStorage.read(newValue, password);

          this.text = text;

      } else {
        this.text = '';
      }

      this.isBusy = false;

    }, {
      immediate: true
    });

    const updateTextStats = debounce(() => {

      let str = this.text;

      if (str) {
        this.wordCount = TextUtil.wordCount(str);
        this.charCount = str.length;
        this.lineCount = TextUtil.lineCount(str);
      }

    }, 100)

    watch(() => this.text, function () {
      updateTextStats();
    }, {
      immediate: true
    });

  },
  async beforeUnmount() {
    //await store.actions.saveContents(this.text);
  }
}
</script>
