<template>
  <section class="container">
    <div  class="login" @click="doLogin">[login:{{user_name}}</div>
    <h1>{{title}}</h1>
    <p class="message">{{message}}</p>
    <div v-if="logined">
    <table>
      <tr>
        <th>Message</th>
        <td><input v-model="msg" size="50"></td>
        <td><button @click="add">投稿</button></td>
      </tr>
    </table>
    <hr>
    <ul v-for="(data, key) in json_data">
      <li><div class="list1">{{data.msg}}</div>
      <div class="list2">{{data.user}} ({{data.posted}})</div>
      </li>
    </ul>
    </div>
  </section>
</template>

<script>
import firebase from 'firebase';

const axios = require('axios');

let config = {
  apiKey: "AIzaSyAbefD18Zm9JdcDTZ5U5Z3xmSIJJPePYWo",
  authDomain: "kaito-vue.firebaseapp.com",
  databaseURL: "https://kaito-vue.firebaseio.com",
  projectId: "kaito-vue",
  storageBucket: "kaito-vue.appspot.com",
  messagingSenderId: "297321245774",
  appId: "1:297321245774:web:087b3e8d0639e2a9b4e71f"
};
firebase.initializeApp(config);

export default {
  data: function() {
    return {
      title: 'Board',
      message: 'ミニ掲示板。最新の投稿を表示します。',
      user_name: '',
      logined: true,
      msg: '',
      page: 0,
      num_per_page: 10,
      json_data: {},
    };
  },
  methods: {
    login: function() {
      let provider = new firebase.auth.GoogleAuthProvider();
      let self = this;
      firebase.auth().signInWithPopup(provider).then(function(result) {
        console.log(result.user);
        self.user = result.user;
        self.user_name = result.user.displayName;
        self.message = 'ログインしました。';
        let db = firebase.database();
        let ref = db.ref('board');
        ref.orderByKey()
                .limitToLast(self.num_per_page)
                .on('value', function(snapshot) {
            if (firebase.auth().currentUser != null) {
                let arr = [];
                let data = snapshot.val();
                for(let item in data) {
                  arr.unshift(data[item]);
                }
                console.log(arr);
                self.json_data = arr;
            } else {
                self.json_data = {};
            }
        });
      });
    },
    logout: function() {
      firebase.auth().signOut();
      this.user_name = '';
      this.json_data = {};
      this.message = 'ログアウトしました。';
    },
    doLogin: function() {
      if (firebase.auth().currentUser == null) {
        this.login();
      } else {
        this.logout();
      }
    },
    add: function() {
      if (firebase.auth().currentUser == null) {
        alert('ログインしないと投稿できません。');
        return;
      }
      let db = firebase.database();
      let user = firebase.auth().currentUser;
      console.log(user);
      let ref = db.ref('board');
      let self = this;
      let d = new Date();
      let dstr = d.getFullYear() + '-' + (d.getMonth() + 1) + '-' + d.getDate() + ' ' + d.getHours() + ':' + d.getMinutes() + ':' + d.getSeconds();
      let id = d.getTime();
      let data = {
        msg: this.msg,
        user: user.displayName,
        posted: dstr,
      };
      firebase.database().ref('board/' + id).set(data);
      this.msg = '';
      this.message = '投稿しました';
    },
  },
  created: function() {
    if (firebase.auth().currentUser == null) {
      this.login();
    }
    console.log(firebase.auth().currentUser);
  }
}
</script>

<style>
.login {
  font-size: 12pt;
  font-weight: bold;
  cursor: pointer;
}
.list {
  text-align: left;
  font-size: 16pt;
}
.list2 {
  text-align: right;
  font-size: 10pt;
}
.container {
  padding: 5px 10px;
}
h1 {
  font-size: 60pt;
  color: #345980;
}
p {
  padding-top: 5px;
  margin: 10px 0px;
  font-size: 20pt;
}
.message {
  font-size: 16pt;
}
div {
  font-size: 14pt;
}
ul {
  margin: 0px 10px;
  background-color: aliceblue;
}
li {
  padding: 10px;
  font-size: 14pt;
}
tr th {
  width: 120px;
  background-color: darkblue;
  color: white;
  font-size: 16pt;
}
tr td {
  padding: 5px 10px;
  background-color: #eef;
  font-size: 14pt;
}
hr {
  margin: 10px 0px;
}
input {
  font-size: 14pt;
}
button {
  font-size: 14pt;
}
</style>