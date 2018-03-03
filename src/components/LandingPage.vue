<template>
    <div>
        <div v-if="ios || macos || android">
            <b-field label="1. Kurs wählen">
                <b-select style="text-align: center" v-model="course" placeholder="Kurs" rounded
                          v-if="Object.keys(courses).length > 0">
                    <option v-for="course in Object.keys(courses)" v-bind:key="course" :value="course">
                        {{ courses[course].title }}
                    </option>
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
            <div v-else-if="course && android">
                <b-field label="2. Synchronisierungsapp aus dem Play Store runterladen">
                    <div>
                        <a href='https://play.google.com/store/apps/details?id=de.yanniks.dhbwlecturescheduler&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img
                                class="gplay" alt='Jetzt bei Google Play'
                                src='../assets/google-play-badge.png'/></a>
                    </div>
                </b-field>
                <b-field label="3. Kalender durch Tipp abonnieren">
                    <div>
                        <button class="button is-primary is-medium"
                                @click="subscribe">
                            abonnieren
                        </button>
                    </div>
                </b-field>
            </div>
            <b-loading :active.sync="isLoading" :canCancel="true"></b-loading>
        </div>
        <div v-else>
            <br>
            Leider funktioniert diese Seite derzeit nur auf macOS und iOS Geräten. Bitte schau später noch einmal vorbei!
        </div>

        <footer>Made with
            <b-icon
                    icon="heart"
                    size="is-small"/>
            by <a href="https://yanniks.de">Yannik Ehlert</a>
            <div v-if="android">
                Google Play und das Google Play-Logo sind Marken von Google LLC.
            </div>
        </footer>
    </div>
</template>

<style scoped>
    .gplay {
        max-width: 200px;
        max-height: 65px;
    }

    footer {
        margin-left: 10px;
        margin-right: 10px;
        margin-top: 30%;
        color: gray;
    }
</style>

<script>
  /* eslint-disable */
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
        if (this.android) {
          // intent://dhbwservices.yanniks.cloud/lectures/wwi2016a#Intent;scheme=webcal;package=de.yanniks.dhbwlecturescheduler;end
          window.location.href = 'intent://dhbwservices.yanniks.cloud/lectures/' +
            this.course + '#Intent;scheme=webcal;package=de.yanniks.dhbwlecturescheduler;end'
          return
        }
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
