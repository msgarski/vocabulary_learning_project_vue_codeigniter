<template>
  <div class="container">
    <div v-if="isLoggedIn" class="list">
      <div v-if="!chooseCourse">
        <div>
          <div class="btn">
            <button
              :disabled="isDisabled"
              @click="setChooseCourse"
              class="button"
            >
              Krótkie powtórki
            </button>
          </div>
        </div>

        <div>
          <div class="btn">
            <button :disabled="isDisabled2" class="button-1">
              Zadania na dzisiaj
            </button>
          </div>
          <div class="btn">
            <router-link to="/mainscreen"
              ><button class="button">Przejdź do programu</button></router-link
            >
          </div>
        </div>
      </div>
      <div v-else class="list-container">
        <ul>
          <fast-repeating
            v-for="course in repeatsForCourses"
            :key="course.course_id"
            :courseId="course.course_id"
            :name="course.name"
            :repAmount="course.repeats"
          >
          </fast-repeating>
        </ul>
        <div class="btn-show">
          <button @click.prevent="goBack()" class="button-1">Wstecz</button>
        </div>
      </div>
    </div>
    <div v-else>
      <p>No i sie zmyło...</p>
    </div>
  </div>
</template>

<script>
import http from "../plugins/axios.js";
import date from "../functions/time/dates.js";
import FastRepeating from "./presentations/FastRepeating.vue";

export default {
  name: "PorchSite",
  components: {
    "fast-repeating": FastRepeating,
  },
  data() {
    return {
      isLogged: this.$store.getters.logInState,
      userId: this.$store.getters.getUserId,
      courseId: 1,
      firstname: "Robert",
      lastname: "Górski",
      isRepeating: false,
      repeatsForCourses: null, // array of courses with repeats
      prawda: "działa",
      chooseCourse: false,
    };
  },
  computed: {
    repeating() {
      return this.$store.getters["repeat/getRepeatsForCourses"];
    },
    isLoggedIn() {
      return this.$store.getters.logInState;
    },
    isDisabled() {
      return !this.isRepeating;
    },
    isDisabled2() {
      return true;
    },
  },
  watch: {
    repeating(newRepeating, oldRepeating) {
      this.repeatsForCourses = newRepeating;
      if (this.repeatsForCourses.length > 0) {
        this.isRepeating = true;
      }
    },
  },
  created() {
    this.setLimitsToStore();
    this.getNumOfRepeatCards();
  },
  methods: {
    getNumOfRepeatCards() {
      let userId = this.$store.getters.getUserId;
      const url = "/repeatQueries/getRepeatsNumsForCourses/" + userId;
      http
        .get(url)
        .then((response) => {
          this.$store.dispatch("repeat/setRepeatsForCourses", response.data);
        })
        .catch((error) => {
          this.errorMessage = error.message;
          console.error("coś poszło nie tak w getNumOfRepeatCards...", error);
        });
    },
    goBack() {
      this.chooseCourse = false;
    },
    setChooseCourse() {
      this.chooseCourse = true;
    },
    setLimitsToStore() {
      http
        .get("options/getOptionsForUser/" + this.userId)
        .then((result) => {
          if (result.data.batch_learning_limit) {
            this.$store.dispatch(
              "option/setLearningBatchLimit",
              result.data.batch_learning_limit
            );
          }
          if (result.data.day_learning_limit) {
            this.$store.dispatch(
              "option/setLearningDayLimit",
              result.data.day_learning_limit
            );
          }
          if (result.data.overlearning) {
            this.$store.dispatch(
              "option/setOverlearning",
              result.data.overlearning
            );
          }
          if (result.data.day_repeat_limit) {
            this.$store.dispatch(
              "option/setRepeatDayLimit",
              result.data.day_repeat_limit
            );
          }
        })
        .catch((error) => {
          this.errorMessage = error.message;
          console.error("coś poszło nie tak...", error);
        });
    },
    getBatchForRepeat() {
      let limit = this.$store.getters["repeat/getRepeatingBatchLimit"];
      const url =
        "/repeatQueries/getRepeatBatchForCourse/" + this.courseId + "/" + limit;

      http
        .get(url)
        .then((response) => {
          this.$store.dispatch("repeat/setBatchForRepeating", response.data);
        })
        .catch((error) => {
          this.errorMessage = error.message;
          console.error("coś poszło nie tak...", error);
        });
    },
    setIsRepeating() {
      this.repeatsForCourses = this.$store.getters[
        "repeat/getRepeatsForCourses"
      ];
      if (this.repeatsForCourses) {
        this.isRepeating = true;
      }
    },
  },
};
</script>

<style scoped>
#title {
  margin-top: 70px;
  text-align: center;
  color: gray;
}
.container {
  width: 50%;
  height: 50%;
  position: absolute;
  border: 1px solid grey(39, 39, 39);
  border-radius: 12px;
  box-shadow: 5px 5px 5px 5px grey;
  margin-top: 180px;
  margin-left: 25%;
  margin-right: auto;
}
.list {
  padding-top: 40px;
}
.list-container {
  position: absolute;
  width: 80%;
  text-align: center;
  margin-top: 40px;
  margin-left: 10%;
  margin-right: 100px;
}
.button {
  background-color: #ffae00; /* Green */
  position: relative;
  border: none;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  height: 40px;
  border-radius: 20px;
  width: 50%;
  margin-right: 5%;
}
.button:hover {
  cursor: pointer;
}
button:disabled,
button[disabled] {
  border: 1px solid #e9e7e7;
  background-color: #e9e7e7;
  color: #c7c7c7;
  cursor: default;
}
.button-1 {
  background-color: #718bff; /* Green */
  position: relative;
  border: none;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  height: 40px;
  border-radius: 20px;
  width: 50%;
  margin-right: 5%;
}
.button-1 {
  cursor: pointer;
}
.buttons {
  width: 100%;
  padding-left: 25%;
  padding-top: 5%;
}
.btn {
  width: 100%;
  text-align: center;
  float: left;
  padding: 20px;
}
.btn-show {
  bottom: 0px;
}

a {
  color: white;
}
</style>
