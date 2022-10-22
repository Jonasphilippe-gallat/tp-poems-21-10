<script setup>
import SignIn from './components/SignIn.vue'
import FamousPoets from './components/FamousPoets.vue'
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = 'https://fhlpskkmjsjwttzhzndg.supabase.co'
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImZobHBza2ttanNqd3R0emh6bmRnIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjI5OTI1NjcsImV4cCI6MTk3ODU2ODU2N30.zo3v04HBOxAwU_QoAESTNxZNMPXfvJ5J_53LLeVTaBA'
const supabase = createClient(supabaseUrl, SUPABASE_KEY)

export default {
  data() {
    return {
      email: 'jophigal@gmail.com',
    }
  }
}

</script>

<template>
  <header>
    <img alt="Poetry" class="logo" src="./assets/logo.png" width="125" height="125" />

    <div class="wrapper">
      <div>
        <SignIn msg="Poet ! Tell us who you are !" />
      </div>
      <label>Enter User name</label><br>
      <input type="email" required name="email" v-model="email"><br>
      <label>Enter Password</label><br>
      <input type="Password" required name="password" v-model="passwd"><br>


      <br><button onClick={createPoem}>Create Poem</button>
      <button>{{email}}</button>


    </div>
  </header>

  <main>
    <FamousPoets />
  </main>
</template>

<script>

import { ref } from "vue"
export default {
  setup() {
    async function createPoem() {
      const { data } = await supabase
        .from('poems')
        .insert([
          { hiden: true, title: 'toto', content: 'titi' }
        ])

      //setPoem({hiden: true, title: "", content: ""})
      //fetchPoems()
    }
    const register = async () => {
      console.log("you are here !")
      const error = await supabase.auth.signUp({
        email: email.value
        //password: passwd.value
      })
    }
  },
}

</script>
<style>
@import './assets/base.css';

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

@media (min-width: 1024px) {
  body {
    display: flex;
    place-items: center;
  }

  #app {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding: 0 2rem;
  }

  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>
