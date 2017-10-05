<template>
  <div id="app">
    <nav class="navbar fixed-top navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="/">Pixels</a>
      <ul class="navbar-nav mr-auto">
        <li class="nav-item" v-for="source in sources" :class="{ active: currentSource == source }">
          <a class="nav-link" href="#" @click.prevent="switchSource(source)">{{ source }}</a>
        </li>
      </ul>
      <ul class="navbar-nav">
        <li class="nav-item">
          <a class="nav-link" href="#" @click.prevent="switchLayout">Layout: {{ layout.multiple ? 'multi' : 'compact' }}</a>
        </li>
      </ul>
    </nav>

    <div class="image-count hidden-xs hidden-sm" @click="getNext" :title="next ? 'Click for more' : ''">
      <div class="count-label" :class="{ clickable: next !== null }">{{ counterText }}</div>
    </div>

    <div class="container">
      <div class="row filters">
        <div class="col-md-2">
          <div class="option" :class="{ active: !isBestOnly }" @click="switchSource(currentSource)">Most recent</div>
        </div>
        <div class="col-md-2" v-for="time in bestOfTimes">
          <div class="option" :class="{ active: isBestOnly && bestOf == time }" @click="showBestOf(time)">Best of {{ time == 'all' ? 'all time' : time }}</div>
        </div>
      </div>

      <div v-if="layout.multiple">
        <template v-for="chunk in chunkedLinks">
          <div class="row">
            <template v-for="link in chunk">
              <div class="col-md-4">
                <post :link="link"></post>
              </div>
            </template>
          </div>
        </template>
      </div>
      <div v-else>
        <template v-for="link in links" track-by="id">
          <div class="row">
            <div :class="(link.preview && link.preview.images[0].source.width) > 980 ? 'col-md-12' : 'col-md-6 col-center'">
              <post :link="link"></post>
            </div>
          </div>
        </template>
      </div>

      <div class="loading-message" v-if="loading">Loading..</div>
      <div class="load-more hidden-md hidden-lg" @click="getNext" v-if="!loading && next !== null">More</div>
    </div>

    <router-view></router-view>
  </div>
</template>

<script>
import Post from './components/Post.vue'

export default {
  name: 'app',
  data () {
    return {
      layout: { multiple: true },
      links: [],
      currentSource: '',
      sources: [
        'MostBeautiful', 'Pixiv', 'touhou', 'RoomPorn', 'Moescape', 'Patchuu',
        'ViewPorn', 'FuturePorn', 'MinimalWallpaper', 'AnimePhoneWallpapers',
        'animewallpaper'
      ],
      bestOfTimes: ['day', 'week', 'month', 'year', 'all'],
      bestOf: 'all',
      isBestOnly: false,
      isNearBottom: false,
      loading: false,
      next: null
    }
  },
  components: {
    Post
  },
  computed: {
    counterText: function() {
      if (this.loading) { return '...'; }
      if (this.isNearBottom) { return 'R'; }
      return this.links.length;
    },
    apiUrl: function() {
      var url = 'https://www.reddit.com/r/' + this.currentSource,
          limit = 30;

      if (this.isBestOnly) {
        url += '/top.json?t=' + this.bestOf + '&limit=' + limit;
      } else {
        url += '.json?limit=' + limit;
      }

      return url;
    },
    chunkedLinks: function() {
      return _.chunk(this.links, 3);
    },
  },
  created: function() {
    this.initLocalData();
    this.fetchData(this.apiUrl);
  },
  methods: {
    initLocalData: function() {
      if (localStorage.getItem('currentSource') === null) {
        this.currentSource = this.sources[0];
        localStorage.setItem('currentSource', this.currentSource);
      } else {
        this.currentSource = localStorage.getItem('currentSource');
      }
      if (localStorage.getItem('multiLayout') === null) {
        localStorage.setItem('multiLayout', this.layout.multiple);
      } else {
        this.layout.multiple = localStorage.getItem('multiLayout') === 'true';
      }
      if (localStorage.getItem('isBestOnly') === null) {
        localStorage.setItem('isBestOnly', this.isBestOnly);
      } else {
        this.isBestOnly = localStorage.getItem('isBestOnly') === 'true';
      }
      if (localStorage.getItem('bestOf') === null) {
        localStorage.setItem('bestOf', this.bestOf);
      } else {
        this.bestOf = localStorage.getItem('bestOf');
      }
    },
    fetchData: function(url) {
      var self = this;
      $.getJSON(url, function(data) {
        self.next = data.data.after;
        var links = _.map(data.data.children, 'data');
        links = _.filter(links, function(link) {
          return link.url.match(/\.(png|jpg|jpeg)$/i) && link.over_18 === false;
        });
        self.links.push(links);
        self.links = _.flatten(self.links);
        self.loading = false;
        self.isNearBottom = false;
        if (self.links.length < 15) {
          self.getNext();
        }
      });
    },
    getNext: function() {
      if (this.next === null) { return; }
      if (this.loading === true) { return; }
      this.loading = true;
      this.fetchData(this.apiUrl + '&after=' + this.next);
    },
    switchSource: function(source) {
      localStorage.setItem('currentSource', source);
      this.currentSource = source;
      this.links = [];
      this.isBestOnly = false;
      localStorage.setItem('isBestOnly', false);
      this.fetchData(this.apiUrl);
    },
    switchLayout: function() {
      this.layout.multiple = !(this.layout.multiple);
      localStorage.setItem('multiLayout', this.layout.multiple);
    },
    showBestOf: function(time) {
      localStorage.setItem('isBestOnly', true);
      this.isBestOnly = true;
      localStorage.setItem('bestOf', time);
      this.bestOf = time;
      this.links = [];
      this.fetchData(this.apiUrl);
    }
  }
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

.col-center
  float: none
  margin: 0 auto

.filters
  line-height: 40px
  color: #fff
  .option
    cursor: pointer
    color: #427fed
    background-color: #fff
    border-radius: 3px
    padding: 0 15px
    margin-bottom: 5px
    box-shadow: 0 1px 0 rgba(0, 0, 0, .1)
    &.active
      color: #fff
      background-color: #427fed

.loading-message
  color: #ccc
  background-color: #f8f8f8
  font-weight: bold
  text-align: center
  border: 1px solid #d8d8d8
  border-radius: 3px
  padding: 10px 0
  margin: 20px 0

.load-more
  @extend .loading-message
  color: #427FED
  cursor: pointer

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

.image-count
  width: 1280px
  margin: 0 auto 0 auto
  .count-label
    transition: top .15s, opacity .15s
    -webkit-user-select: none
    min-width: 50px
    max-width: 75px
    height: auto
    min-height: 60px
    cursor: default
    background-color: #427fed
    box-shadow: 0 1px 0 rgba(0, 0, 0, .05)
    background-image: -webkit-linear-gradient(top, transparent, transparent)
    border: 1px solid transparent
    border-radius: 3px
    color: #fff
    position: fixed
    top: 60px
    font-size: 24px
    padding: 12px 10px
    text-align: center
    &.clickable
      cursor: pointer
</style>
