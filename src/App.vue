<template>
  <v-app id="app">
    <v-container>
      <Editor  v-model="content" />
    </v-container>
  </v-app>
</template>

<script>
import { defineComponent } from '@vue/composition-api'
import Editor from './components/Editor.vue'

export default defineComponent({
  name: 'App',
  components: {
    Editor,
  },
  data() {
    return {
      content: ""
    }
  }
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
