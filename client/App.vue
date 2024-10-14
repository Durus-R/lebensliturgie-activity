<template>
  <div id="app">
    <h1>Lebensliturgien für den {{ new Date().toLocaleDateString() }}</h1>
    <h2>Aktuell in der Alpha-Phase. Nicht verbunden mit lebensliturgien.de</h2>
    <!-- <AVWaveform v-if="audioSrc" :line-width="2" line-color="lime" :src="audioSrc"></AVWaveform>-->
    <br/>
    <div class="player">
      <VueSound v-if="audioSrc"
                :title="title"
                :detais="variation"
                :file="audioSrc"
                :showSkip="false"
      />
    </div>
    <h3 v-if="!audioSrc">Loading...</h3>
    <!--<h1>{{ msg }}</h1>

    <div class="card">
      <button type="button" @click="count++">count is {{ count }}</button>
    </div>-->
    {{ g_error }}
  </div>
</template>

<script setup>
import {computed, onMounted, ref} from 'vue';
import {discordSdk} from "@discord/embedded-app-sdk";
import axios from 'axios';
import {parse} from 'node-html-parser';

import {AVWaveform} from "vue-audio-visual";
import {VueSound} from 'vue-sound';
import {patchUrlMappings} from "@discord/embedded-app-sdk";

const audioSrcLink = ref("")
const audioSrc = ref('');
const count = ref(0);
const msg = ref('Hello, World!');
const g_error = ref('');

const title = computed(() => `Lebensliturgie für den ${new Date().toLocaleDateString()}`);
const variation = ref("Tagesgebet")

async function downloadLlMp3() {
  const day = new Date().getDay();
  const dayMap = {
    0: 'sonntag',
    1: 'montag',
    2: 'dienstag',
    3: 'mittwoch',
    4: 'donnerstag',
    5: 'freitag',
    6: 'samstag'
  };

  if (day !== 0) {
    const hour = new Date().getHours();
    if (hour < 10) {
      variation.value = "Morgengebet"
    } else if (hour < 17) {
      variation.value = "Mittagsgebet"
    } else {
      variation.value = "Abendgebet"
    }
  }
  const baseUrl = `https://lebensliturgien.de/archiv/${dayMap[day]}/${variation.value}`;
  audioSrcLink.value = baseUrl;
  try {
    const response = await axios.get(baseUrl);
    const htmlCode = response.data.toString();
    const parsed = parse(htmlCode);
    const audioTag = parsed.querySelector('audio');
    return audioTag?.getAttribute('src') ?? '';
  } catch (error) {
    g_error.value = error.toString();
    return '';
  }
}

onMounted(async () => {
  patchUrlMappings([{prefix: '/ll', target: 'lebensliturgien.de'}]);
  const mp3_url = await downloadLlMp3();
  let arrayBuffer = await (await fetch(mp3_url)).arrayBuffer();

  const blob = new Blob([arrayBuffer], {type: "audio/wav"});
  audioSrc.value = window.URL.createObjectURL(blob);

});
</script>

<style>
@import "./player.css";

html, body {
  height: 100%;
  margin: 0;
}

body {
  background-image: url('./.proxy/wallpaper.jpg');
  background-repeat: repeat-x;
  background-size: cover;
  background-position: center;
  background-color: rgba(0, 0, 0, 0.65);
  background-blend-mode: darken;
  overflow: hidden;
}

#app {
  margin-top: 18rem;
  text-align: center;
  color: white;
  --player-background: transparent;
  --player-text-color: white;
}

.player {
  max-width: 80%;
  align-items: center;
  margin: auto;
}
</style>