<template>
  <div class="roster-view">
    <v-container fluid class="pa-0 pt-3">
      <v-layout><v-flex class="text-xs-center roster-title"><span>pilot roster</span></v-flex></v-layout>
      <v-tooltip left>
        <v-menu offset-y slot="activator">
          <template v-slot:activator="{ on }">
            <v-btn  icon dark absolute top right style="margin-top: 80px;" v-on="on"><v-icon>mdi-sort</v-icon></v-btn>
          </template>
          <v-list dark dense>
            <v-list-tile v-for="s in sorts" :key="s.name">
              <v-list-tile-title>{{s.name}}</v-list-tile-title>
              <v-spacer class="ml-3"/>
              <v-btn icon class="pa-0 ma-0" @click="sortBy(s, true)"><v-icon color="grey lighten-1">mdi-sort-ascending</v-icon></v-btn>
              <v-btn icon class="pa-0 ma-0" @click="sortBy(s, false)"><v-icon color="grey lighten-1">mdi-sort-descending</v-icon></v-btn>
            </v-list-tile>
          </v-list>
        </v-menu>
        <span>Sort Pilots</span>
      </v-tooltip>
    </v-container>
    <v-container grid-list-xl fluid>
      <v-layout row wrap class="ml-2 mr-2" fill-height>
        <v-flex v-for="(p, i) in pilots" :key="p.id" class="mb-4" xs3>
          <pilot-card :pilot="p" :p-idx="i"/>
        </v-flex>
        <v-flex xs3><add-pilot-card /></v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import _ from 'lodash'
import PilotCard from './PilotCard'
import AddPilotCard from './AddPilotCard'

export default {
  name: 'roster-view',
  components: { PilotCard, AddPilotCard },
  data: () => ({
    sorts: [
      { name: 'Created', field: '' },
      { name: 'Callsign', field: 'callsign' },
      { name: 'Name', field: 'name' },
      { name: 'Background', field: 'background' },
      { name: 'License Level', field: 'level' }
    ],
    currentSort: { name: 'Created', field: '' },
    ascending: false
  }),
  computed: {
    pilots: function () {
      var allPilots = this.$store.getters.getAllPilots
      if (this.currentSort && this.currentSort.field !== '') {
        allPilots = this.currentSort.field === 'level'
          ? _.sortBy(allPilots, this.currentSort.field)
          : _.sortBy(allPilots, p => p[this.currentSort.field].toLowerCase())
      }
      if (!this.ascending) {
        return _.reverse(_.clone(allPilots))
      }
      return allPilots
    }
  },
  methods: {
    sortBy: function (sort, isAscending) {
      this.currentSort = sort
      this.ascending = isAscending
    }
  }
}
</script>

<style>
  .roster-view {
    width: 100%;
    height: 100vh;
    overflow-y: scroll;
    background-color: #424242;
  }

  .roster-title {
    font-size: 35px;
    letter-spacing: 25px;
    font-weight: 100;
    text-transform: uppercase;
    color: #fff;
  }
</style>