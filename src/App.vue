<template>
  <div id="app">    
    <ul class="list">
      <li class="item" v-for="(item, index) in this.items" :key="index">{{item}}</li>
    </ul>
    <infinite-loading v-if="hasNext" @infinite="infiniteHandler" spinner="spiral" direction="bottom">
      <div slot="no-more">No more</div>
      <div slot="no-results">No results</div>
    </infinite-loading>
  </div>
</template>

<script>
import InfiniteLoading from 'vue-infinite-loading'
const axios = require('axios').default;

export default {
  name: 'App',
  components: {
    InfiniteLoading
  },
  data() {
    return {
      items: [],            // リストに表示するデータ
      startPage: 0,         // 開始ページ番号
      endPage: 0,           // 終了ページ番号
      totalPages: 0,        // 総ページ数
      pageSize: 10,         // １ページに表示するデータ件数
      initialized: false    // 初回データアクセスが完了した後にtrueを設定するフラグ
    }
  },
  computed: {
    hasNext() {
      return this.initialized && this.totalPages > this.endPage
    }
  },
  mounted() {

    // 現在表示中のページ番号をURLに設定する為に、スクロールイベントを監視
    window.addEventListener("scroll", () => this.scroll())

    const urlParams = new URLSearchParams(window.location.search);
    const page = urlParams.get('page');

    if (page) {
      // URLパラメータでページ番号が指定された場合、指定ページから表示
      this.startPage = parseInt(page, 10)
      this.endPage = parseInt(page, 10)
    } else {
      // ページ番号の指定がない場合は１ページ目から表示
      this.startPage = 1
      this.endPage = 1
    }

    // 初回データアクセス
    this.getItems(null, this.startPage, false)
  },
  methods: {

    // スクロール時に、次ページに表示するデータを取得する処理
    infiniteHandler($state) {
      if (this.endPage >= this.totalPages) {
        // 表示するデータが無くなったら$state.complete()を呼ぶ
        $state.complete()
      } else {
        // 表示するデータがある場合、時ページのデータを読み込む
        this.getItems($state, this.endPage + 1, true)
      }
    },

    // ページに表示するデータを検索する処理
    getItems($state, page) {
      setTimeout(() => {
      
        // 読込データを設定（実際はaxiosなどで非同期でデータを取得する想定）
        let data = []
        
        axios
          .get('https://api.github.com/repos/facebook/react/issues?page=' + page +'&per_page='+ this.pageSize)
          .then(response => (data = response))
        for (let i = 1 ; i <= this.pageSize; i++) {
          data.push(`item${page}-${i}`)
        }
        // 総ページ数を設定
        this.totalPages = data.length

        // 現在表示しているデータの末尾に取得したデータを追加
        this.items = this.items.concat(data)
        this.endPage = page

        // $state.loaded()でデータの読込完了を通知する
        if ($state) $state.loaded()

        this.$nextTick(() => {
          this.initialized = true
        })
      }, 1000)
    },

    // スクロールイベント発生時の処理
    scroll() {
      // 現在のスクロールY座標から、画面に表示されているページ番号を計算する
      let scroll_pos = window.pageYOffset || document.documentElement.scrollTop
      let window_height = window.outerHeight
      let page = Math.ceil((scroll_pos + 0.5 * window_height) / 40 / this.pageSize) + (this.startPage - 1)
      // replaceStateでurlを書き換え（urlパラメータにページ番号を設定）
      window.history.replaceState(null, null, "/?page=" + page)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.list {
  list-style: none;
  margin: 0;
  padding: 0;
}
.list li {
  height: 40px;
  border-top: solid 1px #ccc;
  border-bottom: solid 1px #777;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
</style>
