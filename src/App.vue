<template>
  <v-app>
    <v-main>
      <MapComponent class="map" />
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

.map {
  position: absolute;
  top: 0px;
  left: 0px;
  z-index: 0;
  width: 100%;
  height: 100%;
}

body {
  font-family: 'SOURCE SANS PRO', REGULAR;
}

</style>
