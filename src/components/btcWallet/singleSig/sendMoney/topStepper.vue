<template>
  <v-stepper :value='currentStep'>
      <v-stepper-header >
        <v-stepper-step step="1" :complete='complete(0)' >Address</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step step="2" :complete='complete(1)' >Amounts</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step step="3" :complete='complete(2)'>Confirm</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step step="4" :complete='complete(3)'>Sign</v-stepper-step>
    </v-stepper-header>
    </v-stepper>
</template>

<script>
export default {
  name: 'topStepper',
  props: ['currentSection'],
  computed: {
    currentStep () {
      const currentStepString = (this.currentSection + 1).toString()
      return currentStepString
    }
  },
  methods: {
    complete (step) {
      const statement = (step < this.currentSection)
      return statement
    }
  }
}
</script>
