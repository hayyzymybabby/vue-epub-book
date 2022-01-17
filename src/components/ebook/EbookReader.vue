<template>
  <div class="ebook-reader">
    <div id="read"></div>
  </div>
</template>

<script>
import { ebookMixin } from '@/utils/mixin'
import Epub from 'epubjs'
export default {
  mixins: [ebookMixin],
  mounted() {
    const fileName = this.$route.params.fileName.split('|').join('/')
    this.setFileName(fileName).then(() => {
      this.initEpub()
    })
  },
  methods: {
    prevPage() {
      if (this.rendition) {
        this.rendition.prev()
        this.setMenuVisible(false)
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next()
        this.setMenuVisible(false)
      }
    },
    toggleTitleAndMenu() {
      this.setMenuVisible(!this.menuVisible)
    },
    initEpub() {
      const url = 'http://192.168.50.236:9001/epub/' + this.fileName + '.epub'
      this.book = new Epub(url)
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight
        // 微信兼容性配置 暂时有无法监听事件的bug
        // method: 'default'
      })
      this.rendition.display()
      this.rendition.on('touchstart', event => {
        this.touchStartX = event.changedTouches[0].clientX
        this.touchStartTime = event.timeStamp
      })
      this.rendition.on('touchend', event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX
        const time = event.timeStamp - this.touchStartTime
        if (time < 500 && offsetX > 40) {
          this.prevPage()
        } else if (time < 500 && offsetX < -40) {
          this.nextPage()
        } else {
          this.toggleTitleAndMenu()
        }
        event.stopPropagation()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.ebook-reader {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>
