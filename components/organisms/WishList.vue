<template>
  <div>
    <Refine
      :display-criteria="bookCriteria"
      :default-criteria="defaultBookCriteria"
    />
    <br /><br />
    <BookList
      :items="items"
      :flag="flag"
      :loading-flag="loadingFlag"
      @wishAddTsundoku="addTsundoku"
      @wishRemoveWishlist="removeWishlist"
      @wishLoadMore="loadMore"
    />
    <BookModal
      :dialog.sync="dialog"
      :item="item"
      title="積み本に追加"
      ok-emit-name="wishlistOk"
      @wishlistOk="ok"
    />
    <RemoveModal
      :dialog.sync="removeDialog"
      :item="removeItem"
      remove-name="気になる本"
      remove-emit-name="wishlistRemove"
      @wishlistRemove="remove"
    />
  </div>
</template>
<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import BookList from '@/components/molecules/BookList.vue'
import BookModal from '~/components/molecules/BookModal.vue'
import RemoveModal from '~/components/molecules/RemoveModal.vue'
import Refine from '@/components/molecules/Refine.vue'
import { WishlistData, SearchData, UpdateWishlistData } from '@/types/book'
import {
  getMoreWishlistData,
  deleteBookDocument,
  updateWishlist
} from '~/api/index'

@Component({
  components: { BookList, Refine, BookModal, RemoveModal }
})
export default class WishList extends Vue {
  @Prop({ default: {} })
  items!: WishlistData[]

  flag: string = 'wishlist'
  item: SearchData = {}
  removeItem: any = {}
  dialog: boolean = false
  removeDialog: boolean = false
  loadingFlag: boolean = false

  bookCriteria: Array<string> = [
    'すべて表示',
    '論文',
    '同人誌',
    '書籍登録が早い順'
  ]
  defaultBookCriteria: string = 'すべて表示'

  addTsundoku(item: any) {
    this.item = item
    this.dialog = true
  }

  removeWishlist(removeItem: any) {
    this.removeItem = removeItem
    this.removeDialog = true
  }
  remove(removeItem: any) {
    deleteBookDocument(removeItem.id)
      .then(() => {
        this.removeItem = {}
        this.$emit('updateWishlist')
        this.$Notice.success({ title: '削除しました。' })
      })
      .catch(error => {
        console.log(error)
      })
  }

  // 積み本に追加する関数
  // TODO: 命名変える
  ok(wishlistData: any) {
    const wishlistTsundokuData: UpdateWishlistData = {
      documentId: wishlistData.item.id,
      currentPageCount: wishlistData.currentPageCount,
      totalPageCount: wishlistData.item.totalPageCount,
      readingStartDate: wishlistData.readingStartDate,
      readingEndExpectedDate: wishlistData.readingEndExpectedDate,
      bookStatus: 'tsundoku'
    }
    updateWishlist(wishlistTsundokuData)
      .then(() => {
        this.$emit('updateWishlist')
        this.$Notice.success({ title: '積み本に追加しました。' })
      })
      .catch(err => {
        console.log(err)
        this.$Notice.error({ title: '追加に失敗しました。' })
      })
  }

  async loadMore(lastBookData: any) {
    this.loadingFlag = true
    await getMoreWishlistData(
      this.$store.state.auth.uid,
      lastBookData.createdAt
    ).then(res => {
      res.docs.map(snapShot => {
        if (snapShot.exists) {
          const data = snapShot.data().items
          data.id = snapShot.id
          data.createdAt = snapShot.data().createdAt
          this.items.push(data)
        }
      })
      this.loadingFlag = false
    })
  }
}
</script>
