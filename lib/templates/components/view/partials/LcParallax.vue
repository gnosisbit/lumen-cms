<template>
  <div :class="currentClass"
       v-bind="currentAttrs"
       v-if="src">
    <v-parallax v-if="currentAttrs && height"
                class="lazyload"
                ref="parallaxElement"
                alt="parallax-image"
                :height="height"
                :jumbotron="$vuetify.breakpoint.smAndDown"
                :src="browserSniffer().isChrome ? '': src"
                :data-prx="src">
      <slot/>
    </v-parallax>
  </div>
</template>

<script>
  import parallaxMixin from '../../../mixins/parallaxMixin'
  import parallaxFixedBgMixin from '../../../mixins/parallaxFixedBgMixing'
  /**
   * @description parallax of vuetify superset with static background
   * IMPORTANT :src must be set otherwise Firefox does not correctly apply transition
   */
  export default {
    name: 'LcParallax',
    mixins: [parallaxMixin, parallaxFixedBgMixin],
    props: {
      defaultHeight: {
        type: Number,
        default: 300
      }
    },
    methods: {
      browserSniffer () {
        if (process.server) return {}
        // https://stackoverflow.com/questions/9847580/how-to-detect-safari-chrome-ie-firefox-and-opera-browser
        // Opera 8.0+
        // eslint-disable-next-line
        const isOpera = (!!window.opr && !!opr.addons) || !!window.opera || navigator.userAgent.indexOf(' OPR/') >= 0

        // Firefox 1.0+
        const isFirefox = typeof InstallTrigger !== 'undefined'

        // Safari 3.0+ "[object HTMLElementConstructor]"
        const isSafari = /constructor/i.test(window.HTMLElement) || (function (p) {
          return p.toString() === '[object SafariRemoteNotification]'
        })(!window['safari'] || (typeof safari !== 'undefined' && safari.pushNotification)) // eslint-disable-line

        // Internet Explorer 6-11
        const isIE = /* @cc_on!@ */false || !!document.documentMode

        // Edge 20+
        const isEdge = !isIE && !!window.StyleMedia

        // Chrome 1+
        const isChrome = !!window.chrome && !!window.chrome.webstore

        // Blink engine detection
        const isBlink = (isChrome || isOpera) && !!window.CSS
        return {isOpera, isFirefox, isSafari, isIE, isEdge, isChrome, isBlink}
      }
    }
  }
</script>

<style lang="stylus">
  [v-jumbotron="true"] {
    img {
      transform: none !important;
      left: 0;
      top: 0;
    }
  }
</style>