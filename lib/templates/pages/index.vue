<template>
  <div class="article-detail-page">
    <lc-content-edit-main v-if="$store.state.lc.isContentEditMode"
                          :page-props="pageProps"
                          :content="pageContent"/>
    <lc-content-renderer v-else-if="pageContent.length"
                         :device="$device"
                         :elements="pageContent"/>
    <div class="content-boxed white elevation-1 pa-3 max-width-700"
         v-if="!pageContent.length && Article && Article.description">
      <h1 v-text="Article.title" class="display-1"/>
      <blockquote v-text="Article.teaser"
                  v-if="Article.teaser"
                  class="my-5"/>
      <div v-html="Article.description"/>
    </div>
  </div>
</template>
<script>
  import _clonedeep from 'clone-deep'
  import ArticleGql from '../gql/article/ArticleBySlug.gql'
  import setPageTemplates from '../util/setPageTemplates'
  import initialAsyncData from '~initialAsyncData'
  import headMetaMixin from '../mixins/headMetaMixin'
  import {GlobalEventBus} from '../util/globalEventBus'

  export default {
    name: 'PageIndex',
    layout: 'pageView',
    mixins: [headMetaMixin],
    middleware: ['auth'],
    head () {
      if (this.Article && this.Article.id) {
        const lang = this.Article.languageKey.toLowerCase()
        return this.getHeadMeta({
          article: this.Article,
          languageKey: lang
        })
      }
      return {}
    },
    data () {
      return {
        Article: null,
        pageProps: null,
        pageContent: []
      }
    },
    mounted () {
      this.scrollToAnchor()
      GlobalEventBus.$on('lc-on-article-content-change', this.onContentChange)
      this.$on('routeChanged', this.onRouteChange)
    },
    beforeDestroy () {
      GlobalEventBus.$off('lc-on-article-content-change', this.onContentChange)
      this.$off('routeChanged', this.onRouteChange)
    },
    // beforeRouteLeave () {
    //   this.onRouteChange()
    // },
    methods: {
      onContentChange () {
        if (this.$apollo.queries.lcArticle) {
          console.log('smart query exists already')
          this.$apollo.queries.lcArticle.refetch()
        } else {
          this.$apollo.addSmartQuery('lcArticle', {
            query: ArticleGql,
            variables () {
              const {slug} = initialAsyncData({store: this.$store, params: this.$route.params, $cms: this.$cms})
              return {slug}
            },
            manual: true,
            result ({data}) {
              console.log('inside smart query', data)
              const article = data.Article
              this.Article = article
              this.pageContent = article.contents
            }
          })
        }
      },
      scrollToAnchor () {
        let hash = this.$route.hash
        if (hash) {
          hash = hash.startsWith('#') ? hash.substr(1) : hash
          const el = document.getElementsByClassName('data-id-' + hash)[0]
          el && this.$vuetify.goTo(el) // el.scrollIntoView()
        }
      },
      onRouteChange () {
        this.$store.dispatch('setPageProps', {})
        this.$store.dispatch('setCurrentArticleCategories', [])
      }
    },
    async asyncData ({req, app, store, params, error, redirect}) {
      const {locale, host, slug} = initialAsyncData({req, store, params, $cms: app.$cms})
      try {
        const apollo = app.apolloProvider.defaultClient
        const res = await Promise.all([
          apollo.query({query: ArticleGql, variables: {slug}})
            .then(({data}) => data),
          setPageTemplates(apollo, store)
        ])
        const data = _clonedeep(res[0])
        const article = data.Article
        const urlAlias = data.UrlAlias
        const articleLang = article && article.languageKey.toLowerCase()
        await store.dispatch('setLanguageKey', articleLang || locale)
        if (article) {
          if (!store.getters.canEdit && (article.deleted || !article.published)) {
            error({
              statusCode: 404,
              message: 'This page could not be found'
            })
          }
          if (app.$cms.languageMustMatch && locale.toUpperCase() !== article.languageKey) {
            error({
              statusCode: 404,
              message: 'This page could not be found'
            })
          }
          await store.dispatch('setPageProps', {
            articleId: article.id,
            languageKey: article.languageKey
          })
          await store.dispatch('setCurrentArticleCategories', article.categories.slice(0))
          return {
            host,
            pageProps: {
              articleId: article.id,
              languageKey: article.languageKey
            },
            Article: article,
            pageContent: article.contents
          }
        } else if (urlAlias && urlAlias.article && urlAlias.article.slug) {
          redirect(301, '/' + urlAlias.article.slug)
        } else {
          error({
            statusCode: 404,
            message: 'This page could not be found'
          })
        }
      } catch (e) {
        console.log('Error', e)
        error({
          statusCode: 500,
          message: 'An error occurred on query the content'
        })
      }
    }
  }
</script>
