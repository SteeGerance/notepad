<template>
    <div class="w-full max-w-screen-lg mx-auto">

        <div style="max-width:700px; margin:3em auto;">

            <input type="text" id="phrase" ref="query" placeholder="Ecrivez-ici votre phrase secrète"
                   autocomplete="off"
                   autocapitalize="off" @keydown.enter="login" v-model="phrase" :disabled="isBusy">


            <p class="my-5">
                <small class="text-muted">Caractères: {{ passLen }}
                    <span class="mx-2"></span> Mots: {{ wordCount }}
                    <span class="mx-2"></span> Entropie: {{ entropy }} bits
                </small>
            </p>

        </div>

    </div>
</template>

<script>

import {TextUtil} from "../classes/TextUtil";
import {Util} from "../classes/Util";

import {Security} from "../classes/Security";
import {APP_KEY} from "../config";
import store from "../store";

export default {
    data() {
        return {
            isBusy: false,
            phrase: '',
        }
    },
    computed: {
        wordCount: function () {
            return TextUtil.wordCount(this.phrase);
        },
        entropy: function () {
            // TODO: do not assume charset size of 26 as most phrases will be [a-z] + space
            return Util.calcEntropy(26, this.phrase.length);
        },
        // words in english dictionary; 171,146 words
        // attacked not at the letter level but at the word level.
        phraseEntropy() {
            return Util.calcEntropy(30000, this.wordCount);
        },
        passLen: function () {
            return this.phrase.length;
        }
    },
    methods: {
        async login() {

            if (this.phrase.length >= 1) {

                this.isBusy = true;

                const val = this.phrase;

                console.time('scrypt');

                const hash = await Security.slowHash(val, APP_KEY);

                console.timeEnd('scrypt');

                this.isBusy = false;

                store.mutations.login(hash);
            }
        }
    },
    mounted() {
        this.$refs.query.focus();
    }
}
</script>
