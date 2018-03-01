<template>
    <div>
        <div v-if="ios || macos || android">
            <b-field label="1. Kurs wählen">
                <b-select style="text-align: center" v-model="course" placeholder="Kurs" rounded v-if="Object.keys(courses).length > 0">
                    <option v-for="course in Object.keys(courses)" v-bind:key="course" :value="course">{{ courses[course].title }}</option>
                </b-select>
                <b-select style="text-align: center" placeholder="Kurs" rounded loading v-else></b-select>
            </b-field>
            <b-field v-if="course && (ios || macos)" :label="secondLabel">
                <div>
                    <button class="button is-primary is-medium"
                            @click="subscribe">
                        abonnieren
                    </button>
                </div>
            </b-field>
            <b-field v-else label="2. Kalender bei Google abonnieren">
                <div>
                    <button class="button is-primary is-medium"
                            @click="subscribe">
                        zu Google Kalender
                    </button>
                </div>
            </b-field>
            <b-loading :active.sync="isLoading" :canCancel="true"></b-loading>
        </div>
        <div v-else>
            <br>
            Leider funktioniert diese Seite derzeit nur auf macOS und iOS Geräten. Bitte schau später noch einmal vorbei!
        </div>

        <footer>Made with <b-icon
                    icon="heart"
                    size="is-small"/> by <a href="https://yanniks.de">Yannik Ehlert</a>
        </footer>
    </div>
</template>

<style scoped>
    footer {
        margin-top: 30%;
        color: gray;
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
      macos: navigator.platform.toUpperCase().indexOf('MAC') >= 0,
      ios: !!navigator.platform && /iPad|iPhone|iPod/.test(navigator.platform),
      android: navigator.userAgent.toLowerCase().indexOf('android') > -1,
      link: 'webcal://' + window.location.host + '/lectures/',
      secondLabel: ''
    }
  },
  beforeMount () {
    this.fetchCourses()
    if (this.macos) {
      this.secondLabel = '2. Kalender durch Klick abonnieren'
    } else {
      this.secondLabel = '2. Kalender durch Tipp abonnieren'
    }
  },
  methods: {
    subscribe () {
      const url = this.link + this.course
      if (this.android) {
        const googleBaseUrl = 'https://calendar.google.com/calendar/r?cid='
        window.location.href = googleBaseUrl + encodeURIComponent(url)
        return
      }
      window.location.href = url
      const vm = this
      vm.isLoading = true
      setTimeout(() => {
        vm.isLoading = false
      }, 10 * 1000)
    },
    fetchCourses () {
      axios.get(window.location.origin + '/courses').then(response => {
        this.courses = response.data
      })
    }
  }
}
</script>
