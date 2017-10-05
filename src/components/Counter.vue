<template>
  <div class="image-count hidden-xs hidden-sm" @click="getNext" :title="next ? 'Click for more' : ''">
    <div class="count-label" :class="{ clickable: next !== null }">{{ counterText }}</div>
  </div>
</template>

<script>
export default {
  name: 'counter',
  props: ['next'],
  computed: {
    counterText: function() {
      if (this.$parent.loading) { return '...'; }
      if (this.$parent.isNearBottom) { return 'R'; }
      return this.$parent.links.length;
    },
  },
  methods: {
    getNext: function() {
      if (this.next === null) { return; }
      if (this.$parent.loading === true) { return; }
      this.$parent.loading = true;
      this.$parent.fetchData(this.$parent.apiUrl + '&after=' + this.next);
    }
  }
}
</script>

<style lang="sass" scoped>
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
