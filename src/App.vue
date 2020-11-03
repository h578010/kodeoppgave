<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">

      <v-icon class="pr-8" x-large color="white">mdi-home</v-icon>

      <v-toolbar-title>Den store lånekalkulatoren</v-toolbar-title>
      </div>

      <v-spacer></v-spacer>

    </v-app-bar>

    <v-main>
      <Kalkulator/>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import Kalkulator from './components/Kalkulator.vue';

export default Vue.extend({
  name: 'App',

  components: {
    Kalkulator,
  },

  data: () => ({
    //
  }),
});
</script>
