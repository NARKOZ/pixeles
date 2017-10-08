<template>
  <div class="post">
    <h4><a class="js-post-link" :href="expandUrl(link.permalink)" :data-id="link.id" target="_blank">{{ link.title }}</a></h4>
    <a class="js-image-link" :href="link.url">
      <img :src="getPreview(link)"
          :data-index="getIndex(link)"
          :data-title="link.title"
          :data-source="link.url"
          :data-source-width="link.preview ? link.preview.images[0].source.width : 0"
          :data-source-height="link.preview ? link.preview.images[0].source.height : 0">
    </a>
  </div>
</template>

<script>
export default {
  name: 'post',
  props: ['link'],
  methods: {
    getIndex: function(link) {
      return this.$parent.links.indexOf(link);
    },
    expandUrl: function(u) {
      return 'https://reddit.com' + u;
    },
    getPreview: function(link) {
      if (link.preview === undefined) { return link.url; }
      var resolution,
          resolutions = link.preview.images[0].resolutions,
          sizes = resolutions.map(function(resolution) {
            return resolution.width;
          });

      if (this.$parent.layout.multiple) {
        if (_.includes(sizes, 640)) {
          resolution = _.filter(resolutions, { 'width': 640 });
        } else {
          resolution = _.filter(resolutions, { 'width': 320 });
        }
      } else {
        if (_.includes(sizes, 1080)) {
          resolution = _.filter(resolutions, { 'width': 1080 });
        } else if (_.includes(sizes, 960)) {
          resolution = _.filter(resolutions, { 'width': 960 });
        } else if (_.includes(sizes, 640)) {
          resolution = _.filter(resolutions, { 'width': 640 });
        } else {
          resolution = _.filter(resolutions, { 'width': 320 });
        }
      }

      if (resolution.length === 0) { return link.url; }
      var previewUrl = _.unescape(resolution[0].url);
      return previewUrl;
    }
  },
}
</script>

<style lang="sass" scoped>
.post
  background-color: #fff
  border-radius: 3px
  border: 1px solid #d8d8d8
  border-bottom-width: 2px
  border-top-width: 0
  padding: 5px
  margin: 10px 0
  img
    width: 100%
    height: auto
    min-height: 100px
  h4
    margin-top: 0
    padding: 10px
    position: absolute
    top: 26px
    left: 34px
    background-color: rgba(0, 0, 0, 0.5)
    border-radius: 2px
    max-width: 85%
    display: none
    a
      color: #fff
    &:hover
      background-color: rgba(0, 0, 0, 0.7)
  &:hover h4
    display: block
</style>
