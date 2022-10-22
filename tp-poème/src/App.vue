<script setup>
import SignIn from './components/SignIn.vue'
import FamousPoets from './components/FamousPoets.vue'
import { createClient } from '@supabase/supabase-js'
import { SupabaseAuthClient } from '@supabase/supabase-js/dist/module/lib/SupabaseAuthClient'
</script>

<template>
  <header>
    <router-link to="/">Go to Home</router-link>
    <img alt="Poetry" class="logo" src="./assets/logo.png" width="125" height="125" />
    <div class="wrapper" id="signOut">
      <div>
        <SignIn msg="Poet ! Tell us who you are !" />
      </div>
      <label>email: </label><br>
      <input type="email" required v-model="email" placeholder="username@domain.tld"><br>
      <label>password: </label><br>
      <input type="password" required v-model="passwd"><br>
      <button v-on:click="register()">Sign Up</button>
      <button v-on:click="login()">Sign In</button>

    </div>
    <div class="hidden" id="addPoem">
      <div>
        <SignIn msg="Write your poem !" />
      </div>
      <h3>The poem remains private, until you make it public</h3>
      <label>Poem's title</label><br>
      <input type="text" required name="title" v-model="title" placeholder="edit me"><br>
      <label>Poem's content</label><br>
      <textarea required name="content" v-model="content" placeholder="edit me" rows="10" cols="50">Your poem here ...
        </textarea> <br>
      <label>Illustration: </label>

      <input type="file" id="file" name="file" placeholder="my file" accept="image/png, image/jpeg"><br>
      <input type="checkbox" v-model="hidden" value=true />
      <label>Hidden poem</label>
      <br><button v-on:click="createPoem()">Add the poem</button>
      <button v-on:click="fetchPoems()">List of poems</button><br>
      <label for="poemtitle" id="poemtitle" style="color: teal;font-weight: 500;"> ... </label>
      <img id="poemillustration" src="./assets/null.jpg" alt="poem illustration" width="75" height="75"
        style="background-color:gray;" /><br>
      <textarea id="poemcontent" readonly rows="10" cols="50"> ... </textarea> <br>
      <button v-on:click="nextPoem()">Next poem</button><br>

      <input type="text" required name="title" v-model="mot" placeholder="Entrez un mot clé"><br>
      <button v-on:click="filtrerPoems()">Filtrage des poems</button><br>

      <label>Poem's language</label><br>
      <input type="text" required name="language" v-model="language" placeholder="edit me" rows="10" cols="10" />
      <label for="poemlanguage" id="poemlanguage" style="color: teal;font-weight: 500;"> ... </label>
      <textarea id="poemlanguage" readonly rows="10" cols="10"> ... </textarea> <br>

    </div>

  </header>

  <main>
    <FamousPoets />
  </main>
</template>

<script>

const SUPABASE_URL = 'https://vyhamgxohivpnbdlbvrg.supabase.co'
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InZ5aGFtZ3hvaGl2cG5iZGxidnJnIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjQ4MDQ4NDgsImV4cCI6MTk4MDM4MDg0OH0.PmFpG7xZ86j0FU3KlPzUx5wMnlTlVeLw1QgVL9gJYqI'
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY)


var poemsList
var currentpoem

export default {
  methods: {
    async register() {
      try {
        const { user, session, error } = await supabase.auth.signUp({
          email: this.email,
          password: this.passwd
        })
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    async login() {
      try {
        const { user, session, error } = await supabase.auth.signIn({
          email: this.email,
          password: this.passwd
        })
        if (error) throw error;
        else {
          document.getElementById('signOut').style.visibility = 'hidden'
          document.getElementById('addPoem').style.visibility = 'visible'
        }
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    async createPoem() {
      var res;

      const { data: objects, error } = await supabase.storage
        .from('images')
        .upload(supabase.auth.user().id + "_" + file.files[0].name, file.files[0])

      res = supabase.storage
        .from('images')
        .getPublicUrl(supabase.auth.user().id + "_" + file.files[0].name).data.publicURL;

      try {
        const { data, error } = await supabase
          .from('poems')
          .insert([
            { hidden: this.hidden, email: this.email, title: this.title, content: this.content, illustrationurl: res }])
        if (error) throw (error)
      } catch (error) { alert(error.error_description || error.meassage) }

    },
    async fetchPoems() {
      try {
        const { data, error } = await supabase
          .from('poems')
          .select()
        poemsList = data
        if (error) throw error;
        if (data.length > 0) {
          document.getElementById('poemtitle').innerHTML = data[0].title + "    "
          document.getElementById('poemcontent').value = data[0].content
          document.getElementById('poemillustration').src = data[0].illustrationurl
        }
        currentpoem = 0;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    //this function allows to display the next accessibe poem for the current user
    //the fetch button should be selected before
    nextPoem() {
      if (currentpoem < poemsList.length - 1) {
        currentpoem++
        document.getElementById('poemtitle').innerHTML = poemsList[currentpoem].title + "    "
        document.getElementById('poemcontent').value = poemsList[currentpoem].content
        document.getElementById('poemillustration').src = poemsList[currentpoem].illustrationurl
      }
    },

    async filtrerPoems() {
      //mange supabase access exceptions
      try {
        //select all accessible poems (owned poems or public ones) 
        const { data, error } = await supabase
          .from('poems')
          .select()
          .like('title', '%' + this.mot + '%')
        poemsList = data
        if (error) throw error;
        //display the first accessible poem if there is at least one poem
        if (data.length > 0) {
          document.getElementById('poemtitle').innerHTML = data[0].title + "    "
          document.getElementById('poemcontent').value = data[0].content
          document.getElementById('poemillustration').src = data[0].illustrationurl
          document.getElementById('poemlanguage').src = data[0].language
        }
        //store the indexof the currently displayed poem
        currentpoem = 0;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },

    nextPoem() {
      if (currentpoem < poemsList.length - 1) {
        currentpoem++
        document.getElementById('poemtitle').innerHTML = poemsList[currentpoem].title + "    "
        document.getElementById('poemcontent').value = poemsList[currentpoem].content
        document.getElementById('poemillustration').src = poemsList[currentpoem].illustrationurl
        document.getElementById('poemlanguage').value = poemsList[currentpoem].language
      }
    }

  }
}
</script>

<style>
@import './assets/base.css';

header .hidden {
  visibility: hidden;
  overflow: hidden;
  display: flex;
  display: inline-block;
  place-items: flex-start;
  flex-wrap: wrap;
}

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
    display: inline-block;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    display: inline-block;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>
