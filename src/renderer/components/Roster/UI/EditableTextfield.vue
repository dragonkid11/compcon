<template>
<div class="pt-1 pb-1 pl-3 pr-3">
  <v-textarea :dark="dark" :color="dark ? 'warning' : 'primary'" box v-model="newVal" v-on:blur="save" auto-grow rows=1 :label="description" clearable>
    {{newVal}}
  </v-textarea>
</div>

</template>

<script>
  export default {
    name: 'editable-textfield',
    props: {
      attr: String,
      description: String,
      initial: String,
      dark: Boolean
    },
    data: () => ({
      newVal: ''
    }),
    methods: {
      save: function () {
        this.$store.dispatch('editPilot', {
          attr: this.attr,
          val: this.newVal
        })
      }
    },
    mounted: function () {
      this.newVal = this.initial
    }
  }
</script>

<style scoped>
  .highlight:hover {
    background-color: aqua;
    cursor: pointer;
  }
</style>