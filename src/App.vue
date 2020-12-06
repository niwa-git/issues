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
      items: [],            // ���X�g�ɕ\������f�[�^
      startPage: 0,         // �J�n�y�[�W�ԍ�
      endPage: 0,           // �I���y�[�W�ԍ�
      totalPages: 0,        // ���y�[�W��
      pageSize: 10,         // �P�y�[�W�ɕ\������f�[�^����
      initialized: false    // ����f�[�^�A�N�Z�X�������������true��ݒ肷��t���O
    }
  },
  computed: {
    hasNext() {
      return this.initialized && this.totalPages > this.endPage
    }
  },
  mounted() {

    // ���ݕ\�����̃y�[�W�ԍ���URL�ɐݒ肷��ׂɁA�X�N���[���C�x���g���Ď�
    window.addEventListener("scroll", () => this.scroll())

    const urlParams = new URLSearchParams(window.location.search);
    const page = urlParams.get('page');

    if (page) {
      // URL�p�����[�^�Ńy�[�W�ԍ����w�肳�ꂽ�ꍇ�A�w��y�[�W����\��
      this.startPage = parseInt(page, 10)
      this.endPage = parseInt(page, 10)
    } else {
      // �y�[�W�ԍ��̎w�肪�Ȃ��ꍇ�͂P�y�[�W�ڂ���\��
      this.startPage = 1
      this.endPage = 1
    }

    // ����f�[�^�A�N�Z�X
    this.getItems(null, this.startPage, false)
  },
  methods: {

    // �X�N���[�����ɁA���y�[�W�ɕ\������f�[�^���擾���鏈��
    infiniteHandler($state) {
      if (this.endPage >= this.totalPages) {
        // �\������f�[�^�������Ȃ�����$state.complete()���Ă�
        $state.complete()
      } else {
        // �\������f�[�^������ꍇ�A���y�[�W�̃f�[�^��ǂݍ���
        this.getItems($state, this.endPage + 1, true)
      }
    },

    // �y�[�W�ɕ\������f�[�^���������鏈��
    getItems($state, page) {
      setTimeout(() => {
      
        // �Ǎ��f�[�^��ݒ�i���ۂ�axios�ȂǂŔ񓯊��Ńf�[�^���擾����z��j
        let data = []
        
        axios
          .get('https://api.github.com/repos/facebook/react/issues?page=' + page +'&per_page='+ this.pageSize)
          .then(response => (data = response))
        for (let i = 1 ; i <= this.pageSize; i++) {
          data.push(`item${page}-${i}`)
        }
        // ���y�[�W����ݒ�
        this.totalPages = data.length

        // ���ݕ\�����Ă���f�[�^�̖����Ɏ擾�����f�[�^��ǉ�
        this.items = this.items.concat(data)
        this.endPage = page

        // $state.loaded()�Ńf�[�^�̓Ǎ�������ʒm����
        if ($state) $state.loaded()

        this.$nextTick(() => {
          this.initialized = true
        })
      }, 1000)
    },

    // �X�N���[���C�x���g�������̏���
    scroll() {
      // ���݂̃X�N���[��Y���W����A��ʂɕ\������Ă���y�[�W�ԍ����v�Z����
      let scroll_pos = window.pageYOffset || document.documentElement.scrollTop
      let window_height = window.outerHeight
      let page = Math.ceil((scroll_pos + 0.5 * window_height) / 40 / this.pageSize) + (this.startPage - 1)
      // replaceState��url�����������iurl�p�����[�^�Ƀy�[�W�ԍ���ݒ�j
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