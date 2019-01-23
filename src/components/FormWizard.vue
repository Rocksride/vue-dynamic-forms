<template>
  <div>
    <div 
      v-if='wizardInProgress'
      v-show='asyncState !== "pending"'
    >
    
      <keep-alive>
        <component 
          ref='currentStep'
          @update='processStep'
          :is='currentStep'
          :wizardData='form'
        ></component>
      </keep-alive>

      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button
          @click="goBack"
          v-if="currentStepNumber > 1"
          class="btn-outlined"
        >Back
        </button>
        <button
          @click="nextButtonAction"
          class="btn"
          :disabled='!canGoNext'
        >{{isLastStep? 'Complete order' : 'Next'}}</button>
      </div>

      <pre><code>{{form}}</code></pre>
    </div>
    <div v-else>
      <h1 class="title">Thank you</h1>
      <h2 class="subtitle">
        We look forward to shipping you your first box!
      </h2>

      <p class="text-center">
        Made by
        <a href="https://github.com/Rocksride" target='_blank' class='btn'>Rocksride</a>
      </p>
    </div>

    <div class="loading-wrapper" v-if='asyncState==="pending"'>
      <div class="loader">
        <!-- <img src="./spinner.svg" alt=""> -->
        <p>Please wait, we're waiting our servers!</p>
      </div>
    </div>
  </div>
</template>

<script>
import { postFormToDB } from '../api'
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      asyncState: null,
      currentStepNumber: 1,
      canGoNext: false,
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      },
      steps: ['FormPlanPicker', 'FormUserDetails', 'FormAddress', 'FormReviewOrder']
    }
  },
  computed: {
    progress () {
      return this.currentStepNumber/this.steps.length * 100
    },
    currentStep() {
      return this.steps[this.currentStepNumber-1]
    },
    isLastStep () {
      return this.currentStepNumber === this.steps.length
    },
    wizardInProgress () {
      return this.currentStepNumber <= this.steps.length
    },
  },
  methods: {
    submitForm () {
      this.asyncState = 'pending'
      postFormToDB(this.form)
        .then(console.log)
        .then(() => {
          this.currentStepNumber++;
          this.asyncState = 'resolved'
        })
        .catch((err) => {
          alert(err);
          this.asyncState = 'rejected'
        })
    },
    nextButtonAction () {
      if (this.isLastStep) {
        this.submitForm()
      } else {
        this.goNext()
      }
    },
    goBack () {
      this.currentStepNumber--
      this.canGoNext = true
    },
    goNext () {
      this.currentStepNumber++
      // this.canGoNext = false;

      //here we could have used this
      // but this requires vuelidate to be plugged in as well as validations: {} property on each component
      // this.$nextTick(() => {
      //   this.canGoNext = !this.$refs.currentStep.$v.$invalid
      // })
      this.$nextTick(() => {
        this.$refs.currentStep.submit()
      })

    },
    processStep ({data, valid}) {
      Object.assign(this.form, data)
      this.canGoNext = valid;
    }
  }
}
</script>
