<template>
  <v-container fluid class="ml-5 pl-5">
    <span class="display-1 text-uppercase font-weight-thin">MANUFACTURERS</span>
    <div class="mt-3">
      <v-tabs v-model="tabModel" color="blue-grey" dark  slider-color="warning" grow>
        <v-tab v-for="m in manufacturers" :key="m.id" ripple >
         {{ m.id }}
        </v-tab>
        <v-tab-item v-for="m in manufacturers" :key="m.id + 'desc'">
          <v-card flat>
            <v-card-title class="display-2 text-uppercase font-weight-light pb-0 mb-0">{{m.name}}</v-card-title>
            <v-card-text class="mt-1 ml-2 pr-4 pt-0"><p v-html="m.description" /></v-card-text>
          </v-card>
        </v-tab-item>
      </v-tabs>
    </div>
  </v-container>
</template>

<script>
  export default {
    name: 'manufacturers',
    data: () => ({
      tabModel: 0,
      manufacturers: {}
    }),
    methods: {
      openItem: function (t) {
        console.error('not yet implemented')
      }
    },
    created: function () {
      this.manufacturers = this.$store.getters.getItemCollection('Manufacturers')
    }
  }
</script>