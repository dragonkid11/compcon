<template>
  <v-card>
    <v-layout v-if="skillData.err">
      <v-card-text class="text-xs-center pa-1">
      <span class="subheading grey--text">// MISSING DATA //</span><br><span v-if="skill.brew" class="caption grey--text">({{skill.brew}})</span>
      </v-card-text>
    </v-layout>
    <v-layout v-else>
      <v-flex xs3>
        <div class="centered text-xs-left pl-3">
          <v-chip dark color="primary" small >
            +<b>{{skill.bonus}}</b>
          </v-chip>
          <span class="subheading font-weight-bold">{{skillData.trigger}}</span>
        </div>
        </v-flex>
      <v-flex xs9>
        <v-expansion-panel class="pa-0 ma-0">
          <v-expansion-panel-content expand-icon="search">
            <template v-slot:header>
              <div><span class="center-align subheading font-weight-regular">{{skillData.description}}</span></div>
            </template>
            <v-card flat class="ma-0 pa-0" width="60vw">
              <v-card-text class="mt-0 mb-0 pt-0 pb-2 blockquote">{{skillData.detail}}</v-card-text>
            </v-card>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-flex>
    </v-layout>
  </v-card>
</template>

<script>
export default {
  name: 'skill-item',
  props: {
    skill: Object,
    skillData: Object
  },
  methods: {
    chipColor: function (skill) {
      if ((skill.specialty && skill.flaw)) return ''
      if (skill.specialty) return 'green'
      else if (skill.flaw) return 'red'
      else return 'primary'
    }
  }
}
</script>

<style scoped>
   .centered {
    position: relative;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
    text-align: center;
  }
</style>

