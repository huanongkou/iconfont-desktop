<style lang="scss">
@include b(page-illustration) {
  @extend .absolute-parent;
  @include e(topbar) {
    z-index: 1;
    @extend .absolute-top;
    @extend .backdrop;
  }
  @include e(bottombar) {
    z-index: 1;
    @extend .absolute-bottom;
    @extend .backdrop;
  }
  @include e(content) {
    @extend .absolute-all;
  }
}
</style>

<template>
  <div class="app-page-illustration">
    <div class="app-page-illustration__content">
      <app-scroll-group ref="list">
        <div slot="header" :style="{ height: ui.topbar.size + 'px' }"/>
        <div slot="footer" :style="{ height: ui.bottombar.size + 'px' }"/>
        <app-dict-select-chip-group>
          <app-dict-select-chip
            v-model="list.query.sort"
            name="collectionSort"
            label="排序"
            @change="listMixinReload"/>
        </app-dict-select-chip-group>
        <app-collection-list :value="list.data">
          <template v-slot="{ collection }">
            <app-collection-mode-search-result :value="collection" illustration/>
          </template>
        </app-collection-list>
      </app-scroll-group>
    </div>
    <div ref="topbar" class="app-page-illustration__topbar pa-4">
      <app-input-search
        class="mx-auto"
        v-model="list.query.keyword"
        placeholder="搜索插画"
        :loading="list.status.isSearching"
        @submit="listMixinReload"/>
    </div>
    <div v-if="list.page.total" ref="bottombar" class="app-page-illustration__bottombar">
      <app-pagination v-model="list.page" @change="listMixinLoadMore"/>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

import mixinPageUI from '@/mixins/page-ui.js'
import mixinPageList from '@/mixins/page-list.js'

export default {
  mixins: [
    mixinPageUI,
    mixinPageList
  ],
  data () {
    return {
      list: {
        query: {
          sort: 'time'
        },
        page: {
          size: 9
        },
        setting: {
          searchWithoutKeywords: true
        }
      }
    }
  },
  computed: {
    ...mapState('sdk', [
      'sdk'
    ])
  },
  mounted () {
    this.listMixinLoad()
  },
  watch: {
    'list.page.total': 'uiLoad'
  },
  methods: {
    async listMixinLoad () {
      this.listMixinScrollTop()
      this.list.data = []
      this.listMixinAddPlaceholder()
      const fetch = this.sdk.collections({
        keyword: this.list.query.keyword,
        sort: this.list.query.sort,
        pageNo: this.list.page.current,
        pageSize: this.list.page.size,
        illustration: true
      })
      const result = await this.listMininFetch(fetch)
      this.listMixinRemovePlaceholder()
      this.list.data = result.lists
      this.list.page.current = result.page
      this.list.page.size = result.limit
      this.list.page.total = result.count
    }
  }
}
</script>
