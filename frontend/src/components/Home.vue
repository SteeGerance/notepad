<template>
    <div class="w-full max-w-screen-lg mx-auto">

        <div style="max-width:700px; margin:3em auto;">
            <h1 class="text-center"> 🔑 Connectez-vous avec une phrase secrète unique</h1>

            <input type="text" id="phrase" ref="query" placeholder="ex: Pregerance, l'IT au profit de votre croissance"
                   autocomplete="off"
                   autocapitalize="off" @keydown.enter="login" v-model="phrase" :disabled="isBusy">

            <div class="text-muted mt-2">Vous pouvez le faire aussi court que vous le souhaitez, mais afin de le rendre plus difficile à deviner pour les autres (ou de force brute), faites-le au moins avec 4 mots.
            </div>

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
