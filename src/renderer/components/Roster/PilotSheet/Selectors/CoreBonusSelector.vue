<template>
  <v-container fluid>
    <v-layout>
      <v-flex xs3>
        <div :class="scrollPosition > 200 ? 'scroll-fix' : ''">
        <v-layout>
          <v-flex style="text-align: center">
          <br>
          <h3>CORE Bonus</h3>
          <hr>
          </v-flex>
        </v-layout>
        <v-layout>
          <v-flex xs12>
            <div v-for="pb in bonuses" :key="`summary_${pb}`">
              <v-layout>
                <v-flex xs12>
                  <strong>{{ bonusById(pb).name }}</strong>&nbsp;<span class="caption">({{ bonusById(pb).source }})</span>
                </v-flex>
              </v-layout>
            </div>
          </v-flex> 
        </v-layout>
        <v-layout><v-flex xs12><hr></v-flex></v-layout>
        <v-layout>
          <v-flex xs12>
            <v-alert outline color="success" icon="check_circle" :value="selectionComplete">
              CORE Bonus Selection Complete
            </v-alert>
            <v-alert outline color="warning" icon="priority_high" :value="!pointLimit">
              {{points.pointsCurrent}} / {{points.pointsMax}} CORE Bonuses selected
            </v-alert>
            <v-btn v-if="!levelUp" block :disabled="!selectionComplete" @click="saveBonuses" color="primary">Save</v-btn>
            <v-btn v-if="!levelUp" block flat small :disabled="!bonuses.length" @click="resetBonuses">Reset</v-btn>
          </v-flex>
        </v-layout>
        </div>
      </v-flex>

      <v-flex xs10 id="list-area">
        <v-layout v-for="bonus in bonusData" :key="bonus.id" >
          <v-flex>
            <v-toolbar>
              <v-toolbar-title>{{bonus.name}}&nbsp;<span class="caption">{{bonus.source}}</span></v-toolbar-title>
              <v-spacer></v-spacer>
              <v-tooltip top :disabled="licenses[bonus.source] > 3 || pointLimit || bonuses.includes(bonus.id)">
                <div slot="activator">
                  <v-btn v-if="bonuses.includes(bonus.id)" fab small @click="removeBonus(bonus.id, bonus.source)"><v-icon>remove</v-icon></v-btn>
                  <v-btn v-else :disabled="pointLimit || licenses[bonus.source] < 3" fab small @click="addBonus(bonus.id, bonus.source)"><v-icon>add</v-icon></v-btn>
                </div>
                <span>{{3 - licenses[bonus.source]}} additional {{bonus.source}} license points required to activate this CORE bonus</span>
              </v-tooltip>
            </v-toolbar>
            <v-card>
              <v-card-title class="pb-0">
                <em v-html="bonus.description" />
              </v-card-title>
              <v-card-text>
                <p v-html="bonus.effect" />
              </v-card-text>
            </v-card>
            <br>
          </v-flex>
        </v-layout>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  // to take a CORE bonus in a manufacturer takes 3 license points, per bonus

  export default {
    name: 'core-bonus-selector',
    props: {
      pilotBonuses: {
        type: Array
      },
      pilotLicenses: {
        type: Array
      },
      pilotLevel: {
        type: Number
      },
      levelUp: {
        type: Boolean
      }
    },
    data: () => ({
      bonuses: [],
      pointLimit: false,
      licenses: {},
      bonusData: [],
      scrollPosition: null
    }),
    computed: {
      points: function () {
        return {
          pointsCurrent: this.bonuses.length,
          pointsMax: Math.floor(this.pilotLevel / 3)
        }
      },
      selectionComplete: function () {
        return this.points.pointsCurrent === this.points.pointsMax
      }
    },
    methods: {
      addBonus: function (id, source) {
        this.bonuses.push(id)
        this.licenses[source] -= 3
        this.pointLimit = this.points.pointsCurrent >= this.points.pointsMax

        if (this.levelUp && this.selectionComplete) {
          this.$emit('set-bonuses', this.bonuses)
          window.scrollTo(0, document.body.scrollHeight)
        }
      },
      removeBonus: function (id, source) {
        var idx = this.bonuses.findIndex(x => x === id)
        if (idx !== -1) {
          this.bonuses.splice(idx, 1)
          this.licenses[source] += 3
        }
        this.pointLimit = false
      },
      saveBonuses () {
        this.$emit('set-bonuses', this.bonuses)
      },
      resetBonuses () {
        this.bonuses = []
        this.$forceUpdate()
        this.pointLimit = false
      },
      bonusById: function (id) {
        return this.$store.getters.getItemById('CoreBonuses', id)
      },
      initialize () {
        var allData = this.$store.getters.getItemCollection('CoreBonuses')
        var licenses = {'GMS': 999}
        for (var i = 0; i < this.pilotLicenses.length; i++) {
          var source = this.pilotLicenses[i].source
          if (licenses[source]) {
            licenses[source] += this.pilotLicenses[i].level
          } else {
            licenses[source] = this.pilotLicenses[i].level
          }
        }
        for (var j = 0; j < this.bonuses.length; j++) {
          var s = allData.find(x => x.id === this.bonuses[j]).source
          licenses[s] -= 3
        }
        this.licenses = licenses
        this.bonusData = allData.filter(x => licenses[x.source] >= 0)
        this.pointLimit = this.points.pointsCurrent >= this.points.pointsMax
      }
    },
    mounted () {
      this.bonuses = JSON.parse(JSON.stringify(this.pilotBonuses))
      this.initialize()
    }
  }
</script>

<style scoped>
  .scroll-fix{
    margin: -25vh 0px;
    position: fixed;
    width: 18vw;
  }

  #list-area {
    width: 80vw!important;
  }

  .center-align {
    min-height: 55px;
    display: inline-flex;
    align-items: center;
  }
</style>
