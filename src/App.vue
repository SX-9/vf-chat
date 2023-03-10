<script setup>
import { createApp, h, ref } from 'vue';
import Mess from './components/Mess.vue';
import swears from './swears.json';
import { initializeApp } from 'firebase/app';
import {
  getAuth,
  signInWithRedirect,
  GithubAuthProvider,
  GoogleAuthProvider,
  signOut,
} from 'firebase/auth';
import {
  getFirestore,
  getDocs,
  addDoc,
  onSnapshot,
  collection,
  orderBy,
  query,
  limit,
} from "firebase/firestore";

initializeApp({
  apiKey: "AIzaSyC17Jru5AC4145DIcoOa5W-cxTm7Phj0CY",
  authDomain: "vf-chat-x.firebaseapp.com",
  projectId: "vf-chat-x",
  storageBucket: "vf-chat-x.appspot.com",
  messagingSenderId: "81255262067",
  appId: "1:81255262067:web:05a65355ac8bfd641bc706"
});

const db = getFirestore();
const msgRefs = collection(db, "messages");
const auth = getAuth();

let bannedWords = swears.concat('http', 'www', 'com');
document.querySelector('title').innerText = 'Vue Firebase Chat';

function isASCII(str, extended) {
  return (extended ? /^[\x00-\xFF]*$/ : /^[\x00-\x7F]*$/).test(str);
}

function msgComponentCreate(msg, user, system) {
  let msgComponent = createApp({ 
    setup () {
      return () => h(Mess, {msg, user, system});
    }
  });

  let wrapper = document.createElement("div");
  msgComponent.mount(wrapper);
  document.querySelector('#messages').appendChild(wrapper);
}

const isAuth = ref(false);
const username = ref('???');
const authToggle = ref(() => {
  if (isAuth.value) {
    signOut(auth).then(() => isAuth.value = false);
  } else {
    let method = prompt(`
      Select A Sign In Method:
        1. Github
        2. Google
      Example: 1
    `);
    let provider;
    if (method == 1) {
      provider = new GithubAuthProvider();
    } else if (method == 2) {
      provider = new GoogleAuthProvider();
    } else {
      alert('Invalid');
      return;
    }
    signInWithRedirect(auth, provider);
    isAuth.value = true;
  }
});

auth.onAuthStateChanged(user => {
  if (user) {
    username.value = user.reloadUserInfo.screenName || user.displayName;
    isAuth.value = true;
  } else {
    username.value = '???';
    isAuth.value = false;
  }
});

const updateMsg = snapshot => {
  document.querySelector('#messages').innerHTML = '';
  snapshot.forEach(doc => {
    msgComponentCreate(
      doc.data().msg, 
      doc.data().author === username.value 
        ? 'You' 
        : doc.data().author, 
      doc.data().author === 'SX-9'
    );
  });
}
const msgQ = query(msgRefs, orderBy('created', 'desc'), limit(10));
onSnapshot(msgQ, updateMsg);

let timeout = false;
const msgCreate = ref(() => {
  if (timeout) return alert('Slowdown, Theres a 5 Second Timeout!');
  timeout = true;
  setTimeout(() => timeout = false, 5000);

  let message = document.querySelector('#msg-input').value;
  if (message.length === 0) return alert('Enter A Message');
  if (!isASCII(message, true)) return alert('Message Contains Non ASCII Characters');
  
  let stop = false;
  bannedWords.forEach(banned => {
    if (stop) return;
    if (message.toLowerCase().includes(banned.toLowerCase())) stop = true;
  });
  if (stop) return alert('Banned Word Detected!');
  
  addDoc(msgRefs, {
    msg: message,
    author: username.value,
    created: Date.now(),
  });

  msgComponentCreate(message, username.value, false);
  message = '';
});

window.onkeypress = e => {
  if (e.key === 'Enter') {
    msgCreate.value();
  }
}
window.addEventListener('DOMContentLoaded', () => msgComponentCreate('Sign In To Chat!', 'Warning', true));
</script>

<template>
<div id="topbar">
  <h1><a href="https://github.com/SX-9/vf-chat">VF-Chat</a></h1>
  <button @click="authToggle">
    <span v-if="isAuth">Log Out</span>
    <span v-else>Log In</span>
  </button>
  <h1 v-if="isAuth" id="username">{{ username }}</h1>
</div>
<div id="messages"></div>
<p id="end">The End!</p>
<div id="inputs" v-if="isAuth">
  <button><a href="#end">????</a></button>
  <input id="msg-input" type="text" placeholder="Hello World, Type Here..."/>
  <button @click="msgCreate">??????</button>
</div>
</template>

<style>
h2, p {
  margin-top: .1rem;
  margin-bottom: .1rem;
}
h1 { margin: .1em; }
a { text-decoration: none; }

#messages { margin-top: 5em; }
#end { text-align: center; padding-bottom: 4em; }

#inputs, #topbar {
  position: fixed;
  left: 0;
  height: 3em;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1em;
}
#topbar { top: 1em; background: #000000a2; }
#topbar h1#username {
  overflow-x: scroll;
  overflow-y: hidden;
  white-space: nowrap;
  max-width: 30%;
}
#topbar h1#username::-webkit-scrollbar {
  height: .1em;
  background-color: white;
  border-radius: 5em;
}
#topbar h1#username::-webkit-scrollbar-thumb {
  background-color: grey;
}

#inputs { bottom: 1em; }
#inputs button { width: 5em; }
#inputs input { width: 60%; }


input, button {
  color: white;
  background: #000000a2;
  border: .1em solid #ffffff;
  padding: 1em;
  border-top-left-radius: .5em;
  border-top-right-radius: .5em;
  border-bottom-left-radius: .5em;
  border-bottom-right-radius: .5em;
}
</style>
