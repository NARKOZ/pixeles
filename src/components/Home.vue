<template>
<div>
  <counter :next="next"></counter>

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
</div>
</template>

<script>
import Counter from './Counter.vue'
import Post from './Post.vue'

export default {
  name: 'home',
  data () {
    return {
      layout: this.$parent.layout,
      links: [],
      currentSource: '',
      sources: this.$parent.sources,
      bestOfTimes: ['day', 'week', 'month', 'year', 'all'],
      bestOf: 'all',
      isBestOnly: false,
      isNearBottom: false,
      loading: false,
      next: null
    }
  },
  components: {
    Counter, Post
  },
  computed: {
    chunkedLinks: function() {
      return _.chunk(this.links, 3);
    },
    apiUrl: function() {
      var subreddit = this.$route.params.subreddit || this.sources[0],
          url = 'https://www.reddit.com/r/' + subreddit,
          limit = 30;

      if (this.isBestOnly) {
        url += '/top.json?t=' + this.bestOf + '&limit=' + limit;
      } else {
        url += '.json?limit=' + limit;
      }

      return url;
    },
  },
  created: function() {
    this.initLocalData();
    this.fetchData(this.apiUrl);
  },
  watch: {
    '$route': 'fetchData'
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
</script>

<style lang="sass" scoped>
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
</style>
