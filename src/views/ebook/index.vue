<template>
  <div class="ebook">
    <EbookTitle></EbookTitle>
    <EbookReader></EbookReader>
    <EbookMenu></EbookMenu>
  </div>
</template>

<script>
import { ebookMixin } from '@/utils/mixin'
import EbookReader from '@/components/ebook/EbookReader'
import EbookTitle from '@/components/ebook/EbookTitle'
import EbookMenu from '@/components/ebook/EbookMenu'
import { getReadTime, saveReadTime } from '@/utils/localStorage'
export default {
  mixins: [ebookMixin],
  components: {
    EbookReader,
    EbookTitle,
    EbookMenu
  },
  mounted() {
    this.startLoopReadTime()
  },
  beforeDestroy() {
    if (this.task) {
      clearInterval(this.task)
    }
  },
  methods: {
    startLoopReadTime() {
      let readTime = getReadTime(this.fileName)
      if (!readTime) {
        readTime = 0
      }
      this.task = setInterval(() => {
        readTime++
        if (readTime % 30 === 0) {
          saveReadTime(this.fileName, readTime)
        }
      }, 1000)
    }
  }
}
</script>

<style lang="scss" scoped></style>
