<template>
  <div class="hello">
    <h2>
      <router-link to="/">Go home</router-link>
    </h2>

    <h3>requestData: {{ requestData }}</h3>
    <button type="button" @click="request">Request</button>
    <br />

    <h1 v-if="!uid"> Hello Recommend items! test </h1>
    <h1 v-if="uid"> Hello Recommend {{ uid }} </h1>

    <h3>text</h3>
    <h3>{{ text }}</h3>
    <input v-model="text">
    <br />
    <button type="button" @click="save">Save</button>

    <div v-for="item in items" :key="item.id">
      <ul>
        <li>text: {{ item.text }}</li>
        <li>date: {{ item.dateStr }}</li>
        <li>id: {{ item.id }}</li>
      </ul>
      <button type="button" @click="deleteData(item.id)">Delete</button>
      <button type="button" @click="recommend(item.text)">Recommend</button>
      <button type="button" @click="predict(item.text)">predict</button>
    </div>

    <h2>Recommend</h2>
    <div v-for="recommend in recommends" :key="recommend.id">
      <p>{{ recommend }}</p>
    </div>
    <h2>Predict</h2>
    <div v-for="pred, predKey in predicts" :key="predKey">
      <p>{{ predKey }}: {{pred}}</p>
    </div>

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

import { getAuth, signInWithEmailAndPassword } from "firebase/auth"
import {
  getFirestore, collection, setDoc, doc, onSnapshot, query, orderBy,
  deleteDoc
  } from "firebase/firestore"
const auth = getAuth()
const db = getFirestore()
import axios from "axios"

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data: () => ({
    text: "",
    email: "",
    password: "",
    isAuthenticated: false,
    uid: false,
    items: [],
    idToken: "",
    requestData: "",
    recommends: [],
    predicts: {}
  }),
  created() {
    console.log("in created:", !!auth.currentUser)
    this.isAuthenticated = !!auth.currentUser
    if (this.isAuthenticated) {
      auth.currentUser.getIdToken(true).then(token => {
        this.idToken = token
        console.log("token", token)
      })
      this.listItems()
    }
  },
  methods: {
    recommend(text) {
      // const url = "http://127.0.0.1:8000/recommend_items"
      // const url = "http://localhost:8000/recommend_items"
      const url = "https://recommend-japq7spw4a-uc.a.run.app/recommend_items"
      const headers = {
        "Authorization" : `Bearer ${this.idToken}`
      }
      const configData = {
        method: "GET",
        url: url,
        headers: headers,
        params: { title: text }
      }
      axios(configData)
      .then(res => {
        this.recommends = res.data.result
      })
    },
    predict(text) {
      // const url = "http://127.0.0.1:8000/predict"
      // const url = "http://localhost:8000/predict"
      const url = "https://recommend-japq7spw4a-uc.a.run.app/predict"
      const headers = {
        "Authorization" : `Bearer ${this.idToken}`
      }
      const configData = {
        method: "GET",
        url: url,
        headers: headers,
        params: { title: text }
      }
      axios(configData)
      .then(res => {
        this.predicts = res.data.result
      })
    },
    request() {
      // const url = "http://127.0.0.1:8000/"
      // const url = "http://localhost:8080/recommend"
      const url = "https://recommend-japq7spw4a-uc.a.run.app"
      const headers = {
        "Authorization" : `Bearer ${this.idToken}`
      }
      const configData = {
        method: "GET",
        url: url,
        headers: headers,
        params: { text: this.text }
      }
      axios(configData)
      .then(res => {
        this.requestData = res.data.result
      })
    },
    deleteData(docId) {
      deleteDoc(doc(db, "recommend", docId))
    },
    toDateTime(seconds) {
      const t = new Date(Date.UTC(1970, 0, 1))
      t.setUTCSeconds(seconds)
      return t
    },
    listItems() {
      const q = query(collection(db, "recommend"), orderBy("date", "desc"))
      onSnapshot(q, (snapShot) => {
        this.items = []
        snapShot.forEach((doc) => {
          const data = doc.data()
          data.id = doc.id
          data.dateStr = this.toDateTime(data.date.seconds).toLocaleString()
          this.items.push(data)
        })
      })
    },
    save() {
      const data = { text: this.text, date: new Date() }
      const path = "recommend"
      const docPath = doc(collection(db, path))
      setDoc(docPath, data)
      .catch(err => {
        console.log("err:", err)
      })
      this.text = ""
    },
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
