<template>
  <h1 id="title">Logowanie</h1>
  <div>
    <form v-if="errorCode == 200" @submit.prevent="authorization" class="form">
      <div>
        <label for="email">E-mail</label>
        <input type="email" name="email" id="email" v-model.trim="email" />
      </div>

      <div>
        <label for="password">Hasło</label>
        <input type="password" name="password" v-model.trim="password" />
      </div>

      <button class="button">Zatwierdź</button>
    </form>
    <div v-else-if="errorCode == 404">
      <p>Nie ma usera</p>
    </div>
    <div v-else-if="errorCode == 401">
      <p>Błąd logowania</p>
    </div>
    <div v-else-if="errorCode == 403">
      <p>Konto nieaktywne</p>
    </div>
  </div>
  <div class="buttons">
    <div class="btn" id="btn-1">
      <button class="button-1"><router-link to="/">Cofnij</router-link></button>
    </div>

    <div class="btn" id="btn-2">
      <button class="button-1">
        <router-link to="/forgot">Zapomniałem hasła...</router-link>
      </button>
    </div>
  </div>
</template>

<script>
import http from "../../plugins/axios.js";

export default {
  name: "SignIn",
  data() {
    return {
      email: "",
      password: "",
      expires_in: "",
      token: "",
      newDate: "",
      errorCode: 200,
    };
  },
  setup() {},
  methods: {
    authorization() {
      const pack = {
        email: this.email,
        password: this.password,
        token: null,
        expires_in: null,
      };
      http
        .post("/login/entering", pack)
        .then((response) => {
          console.log("Pierwsze wywołanie: ", response.data);
          console.dir(response);
          console.log("pierwsze wywołanie ale bez response...");
          this.password = "";
          this.token = response.data.token;

          localStorage.setItem("token", this.token);
          this.$store.dispatch("setUserId", response.data.userId);
          this.$store.dispatch("login");
          console.log("drugie wywołanie: ", this.$store.getters.logInState);
          console.log("userId: ", this.$store.getters.getUserId);
          this.$store.dispatch("setTodayDate");
        })
        .then(() => {
          this.$router.push("/porch");
        })
        .catch((error) => {
          switch (error.response.status) {
            case 403:
              this.errorCode = 403; // not getting here
              break;
            case 404:
              this.errorCode = 404; // or here
              break;
            case 401:
              this.errorCode = 401; // or here
              break;

            default:
              console.log("some other error");
              break;
          }
        });
    },
  },
};
</script>

<style scoped>
.container {
  border: 1px solid grey(39, 39, 39);
  border-radius: 12px;
  box-shadow: 5px 5px 5px 5px grey;
  margin-top: 20px;
  margin-left: 100px;
  margin-right: 100px;
  height: 400px;
}
#title {
  margin-top: 70px;
  text-align: center;
  color: gray;
}
.form {
  margin-top: 100px;
  padding-left: 40%;
}
label {
  display: block;
  font-size: 16px;
  font-style: italic;
}
input {
  width: 40%;
  padding: 12px 20px;
  margin: 8px 0;
  box-sizing: border-box;
  border-radius: 5px;
}

.button {
  background-color: #ffae00; /* Green */
  border: none;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  width: 100px;
  height: 40px;
  border-radius: 20px;
}
.buttons {
  width: 100%;
  padding-left: 25%;
  padding-top: 5%;
}
.button-1 {
  background-color: #718bff; /* Green */
  border: none;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  height: 40px;
  border-radius: 20px;
}
.btn {
  width: 20%;
  text-align: center;
  float: left;
  padding: 20px;
}

a {
  color: white;
}
</style>
