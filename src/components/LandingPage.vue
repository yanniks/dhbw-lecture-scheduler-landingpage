<template>
    <div>
        <div v-if="ios || macos">
            <b-field label="1. Kurs wählen">
                <b-select style="text-align: center" v-model="course" placeholder="Kurs" rounded v-if="Object.keys(courses).length > 0">
                    <option v-for="course in Object.keys(courses)" v-bind:key="course" :value="course">{{ courses[course].title }}</option>
                </b-select>
                <b-select style="text-align: center" placeholder="Kurs" rounded loading v-else></b-select>
            </b-field>
            <b-field v-if="course" :label="secondLabel">
                <div>
                    <button :href="link + course" class="button is-primary is-medium"
                            @click="subscribe">
                        abonnieren
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
      window.location.href = this.link + this.course
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
