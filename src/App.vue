<template>
  <div id="app">
    <nav class="navbar fixed-top navbar-expand-lg navbar-light bg-light" @click="scrollToTop">
      <router-link class="navbar-brand" to="/">Pixels</router-link>

      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item" v-for="source in sources" :class="{ active: $route.params.subreddit == source }">
            <router-link class="nav-link" :to="{ name: 'subreddit', params: { subreddit: source } }">{{ source }}</router-link>
          </li>
        </ul>
        <ul class="navbar-nav">
          <li class="nav-item">
            <a class="nav-link" href="#" @click.prevent="switchLayout">Layout: {{ layout.multiple ? 'multi' : 'compact' }}</a>
          </li>
        </ul>
      </div>
    </nav>

    <transition>
      <router-view :key="$route.fullPath"></router-view>
    </transition>
  </div>
</template>

<script>
import Home from './components/Home.vue'

export default {
  name: 'app',
  data () {
    return {
      layout: { multiple: true },
      sources: [
        'MostBeautiful', 'Pixiv', 'RoomPorn', 'ViewPorn', 'FuturePorn', 'Moescape', 'MinimalWallpaper',
        'AnimePhoneWallpapers', 'animewallpaper'
      ],
    }
  },
  components: {
    Home
  },
  methods: {
    switchLayout: function() {
      this.layout.multiple = !(this.layout.multiple);
      localStorage.setItem('multiLayout', this.layout.multiple);
    },
    scrollToTop: function() {
      var $body = $('html, body');
      $body.animate({ scrollTop: 0 }, 'fast');
    }
  },
}

var pswpElement = $('.pswp')[0],
    galleryOptions = {
      history: false,
      preload: [1, 2]
    },
    gallery;

function galleryItems() {
  var images = $('.post img'),
      items = [],
      $img, item;

  _.forEach(images, function(img) {
    $img = $(img);
    item = {
      src: $img.data('source'),
      w: $img.data('source-width') || $img.width(),
      h: $img.data('source-height') || $img.height(),
      msrc: $img.attr('src'),
      title: $img.data('title')
    };
    items.push(item);
  });

  return items;
}

$('body').on('click', '.js-image-link', function(e) {
  e.preventDefault();
  var items = galleryItems(),
      shareUrl = 'https://redd.it/' + $(this).parent().find('.js-post-link').data('id');
  galleryOptions.index = $(this).find('img').data('index');
  galleryOptions.getPageURLForShare = function(shareButtonData) { return shareUrl; };
  gallery = new PhotoSwipe(pswpElement, PhotoSwipeUI_Default, items, galleryOptions);
  gallery.init();
});
</script>

<style lang="sass">
@import "~bootstrap/scss/bootstrap"
@import "~photoswipe/dist/photoswipe.css"
@import "~photoswipe/dist/default-skin/default-skin.css"

body
  padding: 60px 0 20px 0
  background-color: #e5e5e5
  font-family: 'Roboto', Arial, sans-serif

.navbar
  height: 42px
  background-color: #fff !important
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, .15)
  .nav-item
    a
      color: #737373 !important
    &.active
      background-color: #f5f5f5

.navbar-nav
  text-align: center
  float: none
  li
    float: none
    display: inline-block
    margin: 0 5px
    border-bottom: 2px solid transparent
    transition: border-color .218s
    &:hover
      border-color: #427fed
      transition: border-color .218s
    &.active
      color: #262626
      font-weight: bold
      border-bottom: 2px solid #427fed
</style>
