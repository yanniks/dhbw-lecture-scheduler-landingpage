<template>
  <div>
    <div style="margin: 5px">
      <b-field
        v-if="courseMethod == 0"
        label="Verwendet dein Kurs eine PDF-Datei oder greifst du über eine Website (Rapla) auf deinen Vorlesungsplan zu?"
      >
        <div>
          <button
            style="margin:10px"
            class="button is-primary is-medium"
            @click="rapla(false)"
          >PDF-Datei</button>
          <button
            style="margin:10px"
            class="button is-primary is-medium"
            @click="rapla(true)"
          >Website</button>
        </div>
      </b-field>
      <b-field
        v-if="courseMethod == 1"
        label="Bitte füge deinen Rapla-Link in das untenstehende Textfeld ein."
      >
        Alternativ reicht es auch, nur den Rapla-Key einzufügen.
        <div style="text-align: center;margin:30px">
          <span style="max-width: 300px;margin:10px">
            <b-field
              :type="!raplaKey && originalRaplaUrl ? 'is-danger' : ''"
              :message="!raplaKey && originalRaplaUrl ? 'Dieser Rapla-Link ist nicht gültig' : ''"
            >
              <b-input
                autocomplete="off"
                autocorrect="off"
                autocapitalize="off"
                v-model="originalRaplaUrl"
                @input="raplaUrlChanged"
                placeholder="https://rapla.dhbw-stuttgart.de/rapla?key=abcdef"
              />
            </b-field>
          </span>
        </div>
      </b-field>
      <b-field v-if="courseMethod == 2" label="1. Kurs wählen">
        <b-select
          style="text-align: center"
          v-model="course"
          placeholder="Kurs"
          rounded
          v-if="Object.keys(courses).length > 0"
        >
          <option
            v-for="course in Object.keys(courses)"
            v-bind:key="course"
            :value="course"
          >{{ courses[course].title }}</option>
        </b-select>
        <b-select style="text-align: center" placeholder="Kurs" rounded loading v-else></b-select>
      </b-field>
      <b-field v-if="(course || raplaKey) && (ios || macos)" :label="secondLabel">
        <div>
          <button class="button is-primary is-medium" @click="subscribe">abonnieren</button>
        </div>
      </b-field>
      <div class="urlInputField" v-else-if="(course || raplaKey) && (!macos && !ios)">
        <b-field
          label="2. Dein Betriebssystem wird leider nicht nativ unterstützt. Bitte importiere folgenden iCal-Link in eine Synchronisierungsanwendung deiner Wahl:"
        >
          <div style="text-align: center">
            <div class="boxedUrlLink">
              <b-input disabled="true" type="text" :value="fullSubscriptionUrl"/>
              <b-icon
                style="margin-left: 10px"
                v-clipboard:copy="fullSubscriptionUrl"
                icon="content-copy"
                size="is-medium"
              />
            </div>
          </div>
        </b-field>
      </div>
      <b-loading :active.sync="isLoading" :canCancel="true"></b-loading>
    </div>

    <footer>
      Made with
      <b-icon icon="heart" size="is-small"/> by
      <a href="https://yanniks.de">Yannik Ehlert</a>
    </footer>
  </div>
</template>

<style scoped>
.gplay {
  max-width: 200px;
  max-height: 65px;
}

.boxedUrlLink {
  margin: 30px;
  display: inline-flex;
}

.urlInputField {
  text-align: center;
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
import axios from "axios";

export default {
  data() {
    return {
      // 0 = undecided, 1 = rapla, 2 = PDF
      courseMethod: 0,
      isLoading: false,
      courses: {},
      course: "",
      originalRaplaUrl: "",
      raplaKey: undefined,
      macos: navigator.platform.toUpperCase().indexOf("MAC") >= 0,
      ios: !!navigator.platform && /iPad|iPhone|iPod/.test(navigator.platform),
      android: navigator.userAgent.toLowerCase().indexOf("android") > -1,
      link: "webcal://lectures.yanniks.app/lectures",
      secondLabel: ""
    };
  },
  beforeMount() {
    this.fetchCourses();
    if (this.macos) {
      this.secondLabel = "2. Kalender durch Klick abonnieren";
    } else {
      this.secondLabel = "2. Kalender durch Tipp abonnieren";
    }
  },
  computed: {
    fullSubscriptionUrl: function() {
      if (this.raplaKey) {
        return `${this.link}?key=${this.raplaKey}`;
      }
      return `${this.link}/${this.course}`;
    }
  },
  methods: {
    subscribe() {
      if (this.android) {
        // intent://dhbwservices.yanniks.cloud/lectures/wwi2016a#Intent;scheme=webcal;package=de.yanniks.dhbwlecturescheduler;end
        window.location.href =
          "intent://dhbwservices.yanniks.cloud/lectures/" +
          this.course +
          "#Intent;scheme=webcal;package=de.yanniks.dhbwlecturescheduler;end";
        return;
      }
      window.location.href = this.fullSubscriptionUrl;
      const vm = this;
      vm.isLoading = true;
      setTimeout(() => {
        vm.isLoading = false;
      }, 10 * 1000);
    },
    /// Returns the query variable from a passed URL object
    getQueryVariable(url, variable) {
      if (!url.search || url.search.substr(0, 1) !== "?") {
        return undefined;
      }
      var query = url.search.substring(1);
      var vars = query.split("&");
      for (var i = 0; i < vars.length; i++) {
        var pair = vars[i].split("=");
        if (decodeURIComponent(pair[0]) == variable) {
          return decodeURIComponent(pair[1]);
        }
      }
      return undefined;
    },
    // Called in case the passed URL changed. Sets the raplaKey variable to the correct new value.
    raplaUrlChanged(newVal) {
      const trimmedUrl = newVal.trim();
      let key = undefined;
      if (
        trimmedUrl.substr(0, 37) === "https://rapla.dhbw-stuttgart.de/rapla" ||
        trimmedUrl.substr(0, 36) === "http://rapla.dhbw-stuttgart.de/rapla"
      ) {
        const url = new URL(trimmedUrl);
        const parsedKey = this.getQueryVariable(url, "key");
        if (parsedKey !== undefined && parsedKey.length >= 64) {
          key = parsedKey;
        }
      }
      if (trimmedUrl.indexOf("&") === -1 && trimmedUrl.length >= 64) {
        key = trimmedUrl;
      }
      console.log(`New key value: ${key}`);
      this.raplaKey = key;
    },
    // Called after first selection button press.
    rapla(usesRapla) {
      if (usesRapla) {
        this.courseMethod = 1;
      } else {
        this.courseMethod = 2;
      }
    },
    fetchCourses() {
      axios.get("https://lectures.yanniks.app/courses").then(response => {
        this.courses = response.data;
      });
    }
  }
};
</script>
