<template>
  <div class="hello">
    <h2>
      <router-link to="/recommend">Go to recommend</router-link>
    </h2>

    <h1 v-if="!uid"> Hello User! </h1>
    <h1 v-if="uid"> Hello {{ uid }} </h1>
    <div v-if="!isAuthenticated">
      <p>Email</p>
      <input type="text" v-model="email">
      <p>Password</p>
      <input type="password" v-model="password">
      <br />
      <button type="button" @click="login">Login</button>
    </div>
    <div v-if="isAuthenticated">
      <button type="button" @click="logout">Logout</button>
    </div>
  </div>
</template>

<script>

import { initializeApp } from "firebase/app";
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";

const firebaseConfig = {
  apiKey: process.env.VUE_APP_apiKey,
  authDomain: process.env.VUE_APP_authDomain,
  projectId: process.env.VUE_APP_projectId,
  storageBucket: process.env.VUE_APP_storageBucket,
  messagingSenderId: process.env.VUE_APP_messagingSenderId,
  appId: process.env.VUE_APP_appId,
  measurementId: process.env.VUE_APP_measurementId
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data: () => ({
    email: "",
    password: "",
    isAuthenticated: false,
    uid: false
  }),
  methods: {
    login() {
      signInWithEmailAndPassword(
        auth,
        this.email,
        this.password
      ).then(user => {
        this.uid = user.user.uid
        console.log("uid", user.user.uid)
        console.log("email", user.user.email)
        console.log("auth", !!auth.currentUser)
        this.isAuthenticated = !!auth.currentUser
      })
    },
    logout() {
      auth.signOut()
      .then(() => {
        console.log("logOut_auth:", !!auth.currentUser)
        this.isAuthenticated = !!auth.currentUser
        this.uid = false
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
