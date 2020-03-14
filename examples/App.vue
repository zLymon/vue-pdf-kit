<template>
  <div id="app">
    <Pdf
      :sourceData="url"
      :is-loaded.sync="isLoaded"
      @handlePageData="data => pageData = data" />
      <Page
        v-for="(page, index) in pageData"
        :page="page"
        :scale="scale"
        :key="index"
        :is-render-text="true" />
  </div>
</template>

<script>
import Pdf from '~/Core/Pdf'
import Page from '~/Core/Page'

export default {
  name: 'App',
  components: {
    Pdf,
    Page
  },
  data () {
    return {
      url: '',
      isLoaded: false,
      pageData: [],
      scale: 1
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
