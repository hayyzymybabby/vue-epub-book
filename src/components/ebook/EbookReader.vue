<template>
  <div class="ebook-reader">
    <div id="read"></div>
  </div>
</template>

<script>
import { ebookMixin } from '@/utils/mixin'
import {
  getFontFamily,
  saveFontFamily,
  getFontSize,
  saveFontSize
} from '@/utils/localStorage'
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
        this.hideTitleAndMenu()
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next()
        this.hideTitleAndMenu()
      }
    },
    toggleTitleAndMenu() {
      if (this.menuVisible) {
        this.setSettingVisible(-1)
        this.setFontFamilyVisible(false)
      }
      this.setMenuVisible(!this.menuVisible)
    },
    hideTitleAndMenu() {
      this.setMenuVisible(false)
      this.setSettingVisible(-1)
      this.setFontFamilyVisible(false)
    },
    initFontSize() {
      const fontSize = getFontSize(this.fileName)
      if (!fontSize) {
        saveFontSize(this.fileName, this.defaultFontSize)
      } else {
        this.setDefaultFontSize(fontSize)
        this.currentBook.rendition.themes.fontSize(fontSize + 'px')
      }
    },
    initFontFamily() {
      const font = getFontFamily(this.fileName)
      if (!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily)
      } else {
        this.setDefaultFontFamily(font)
        this.currentBook.rendition.themes.font(font)
      }
    },
    initEpub() {
      const url =
        `${process.env.VUE_APP_RES_URL}/epub/` + this.fileName + '.epub'
      this.book = new Epub(url)
      this.setCurrentBook(this.book)
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight,
        allowScriptedContent: true
        // 微信兼容性配置 暂时有无法监听事件的bug
        // method: 'default'
      })
      this.rendition.display().then(() => {
        this.initFontSize()
        this.initFontFamily()
      })
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
      this.rendition.hooks.content.register(contents => {
        Promise.all([
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/cabin.css`
          )
        ]).then(() => {})
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
