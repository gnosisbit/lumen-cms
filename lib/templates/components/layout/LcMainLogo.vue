<template>
  <img :src="path || ($vuetify.breakpoint.lgAndUp ? logoPath : logoMobilePath)"
       height="100%"
       width="auto"
       alt="logo"
       @click="onLogoClick"
       class="toolbar-logo">
</template>

<script>
  // import CONFIG from '../../../src/config'

  export default {
    name: 'LcMainLogo',
    props: {
      path: {
        type: String | null
      }
    },
    computed: {
      logoPath () {
        return this.path || this.$cms.logoPath
      },
      logoMobilePath () {
        // fallback is logoPath
        return this.$cms.logoPathMobile || this.$cms.logoPath
      }
    },
    methods: {
      onLogoClick () {
        const route = this.$cms.defaultLanguage === this.$store.state.lc.locale ? '/' : '/' + this.$store.state.lc.locale
        this.$router.push(route)
      }
    }
  }
</script>

<style scoped>
  .toolbar-logo {
    height: 40px;
    cursor: pointer;
  }
</style>
