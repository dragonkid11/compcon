<template>
  <div>
    <slot name="label"></slot>
    <v-dialog width="600" v-model="dialog">
      <v-btn slot="activator" class="edit-btn mr-0 pr-0" right small flat icon :color="dark ? 'amber accent-3' : 'primary'"><v-icon small>edit</v-icon></v-btn>
      <v-card>
        <v-card-title class="headline" primary-title>Edit {{description}}</v-card-title>
        <v-card-text>
          <v-text-field v-model="newLabel" :label="description" :placeholder="placeholder" type="text" clearable></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn flat @click="dialog = false"> Cancel </v-btn>
          <v-spacer></v-spacer>
          <v-btn :disabled="!newLabel || !newLabel.length || newLabel === ''" color="primary" flat @click="save(attr)">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  export default {
    name: 'editable-label',
    props: {
      attr: String,
      description: String,
      placeholder: String,
      dark: Boolean
    },
    data: () => ({
      dialog: false,
      newLabel: ''
    }),
    methods: {
      save: function (attr) {
        this.$store.dispatch('editPilot', {
          attr: attr,
          val: this.newLabel
        })
        this.dialog = false
        this.$emit('on-save')
      }
    }
  }
</script>