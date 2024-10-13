<template>
  <div id="app">
    <h1>Nicht verbunden mit dem offiziellen Anbieter!</h1>
    <h2>Aktuell in der Alpha-Phase.</h2>
    <av-line v-if="audioSrc" :line-width="2" line-color="lime" :src="audioSrc"></av-line>
    <h1>{{ msg }}</h1>

    <div class="card">
      <button type="button" @click="count++">count is {{ count }}</button>
    </div>
    {{ g_error }}
  </div>
</template>

<script setup>
import {onMounted, ref} from 'vue';
import axios from 'axios';
import {parse} from 'node-html-parser';
import VueAudio from 'vue-audio';
import {patchUrlMappings} from "@discord/embedded-app-sdk";

const audioSrc = ref('');
const count = ref(0);
const msg = ref('Hello, World!');
const g_error = ref('');

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
  const baseUrl = `https://lebensliturgien.de/archiv/${dayMap[day]}/tagesgebet`;

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

  const blob = new Blob([arrayBuffer], { type: "audio/wav" });
  audioSrc.value = window.URL.createObjectURL(blob);
});
</script>

<style>
#app {
  text-align: center;
}
</style>