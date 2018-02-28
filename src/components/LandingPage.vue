<template>
    <div>
        <div v-if="ios">
            <b-field label="1. Kurs wählen">
                <b-select v-model="course" class="selector" placeholder="Kurs" rounded v-if="Object.keys(courses).length > 0">
                    <option v-for="course in Object.keys(courses)" v-bind:key="course" :value="course">{{ courses[course].title }}</option>
                </b-select>
                <b-select class="selector" placeholder="Kurs" rounded loading v-else></b-select>
            </b-field>
            <b-field v-if="course" label="2. Kalender durch klick abonnieren">
                <div>
                    <button :href="link + course" class="button is-primary is-medium"
                            @click="subscribe">
                        Abonnieren
                    </button>
                </div>
            </b-field>
            <b-loading :active.sync="isLoading" :canCancel="true"></b-loading>
        </div>
        <div v-else>
            <br>
            Leider funktioniert diese Seite derzeit nur auf iOS Geräten. Bitte schau später noch einmal vorbei!
        </div>
    </div>
</template>

<style>
    .selector {
        text-align: center;
    }
</style>
<script>
import axios from 'axios'

export default {
  data () {
    return {
      isLoading: false,
      courses: {},
      course: '',
      ios: !!navigator.platform && /iPad|iPhone|iPod/.test(navigator.platform),
      link: 'webcal://dhbwservices.yanniks.cloud/lectures/'
    }
  },
  beforeMount () {
    this.fetchCourses()
  },
  methods: {
    subscribe () {
      window.location.href = this.link + this.course
      const vm = this
      vm.isLoading = true
      setTimeout(() => {
        vm.isLoading = false
      }, 10 * 1000)
    },
    fetchCourses () {
      axios.get('https://dhbwservices.yanniks.cloud/courses').then(response => {
        this.courses = response.data
      })
    }
  }
}
</script>
