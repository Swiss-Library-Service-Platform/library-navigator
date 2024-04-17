<template>
  <v-app>
    <v-main>
      <MapComponent />
    </v-main>
  </v-app>
</template>

<script>
import MapComponent from './components/MapComponent.vue';

export default {
  name: 'App',

  components: {
    MapComponent,
},

  data: () => ({
    return: {
    }
  }),
  methods: {
  }
};
</script>

<style>
html {
  overflow-y: hidden !important;
}

body {
  font-family: 'SOURCE SANS PRO', REGULAR;
}

</style>
