<template>
  <div>
    <Video :src="videoUrl" :poster="imageUrl"  width="800" height="500"/>
  </div>
</template>

<script>
import Video from './components/video.vue'


export default {
  name: 'App',
  components: {
    Video
  },
  setup() {
    const videoUrl = new URL('./assets/video/online.mp4', import.meta.url).href
    const imageUrl = new URL('./assets/header.jpg', import.meta.url).href
    return {
      videoUrl,
      imageUrl
    }
  }
}
</script>
