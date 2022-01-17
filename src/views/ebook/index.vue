<template>
  <div class="ebook">
    <EbookTitle></EbookTitle>
    <EbookReader></EbookReader>
    <EbookMenu></EbookMenu>
  </div>
</template>

<script>
import EbookReader from '@/components/ebook/EbookReader'
import EbookTitle from '@/components/ebook/EbookTitle'
import EbookMenu from '@/components/ebook/EbookMenu'
export default {
  components: {
    EbookReader,
    EbookTitle,
    EbookMenu
  }
}
</script>

<style lang="scss" scoped></style>
