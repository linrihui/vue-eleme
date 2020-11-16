<template>
  <div id="app">
    <router-view/>
    <footer-guide/>
  </div>
</template>

<script>
import FooterGuide from './components/FooterGuide/FooterGuide'
export default {
  components:
    {
      FooterGuide
    }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  #app
    width 100%
    height 100%
    background #F5F5F5
</style>
