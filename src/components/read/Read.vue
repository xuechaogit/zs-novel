<template>
  <div>
    <x-header :left-options="{preventGoBack: true}" :right-options="{showMore: true}" @on-click-back="clickBack" @on-click-more="showMenus = true">{{title}}</x-header>
    <div class="content" v-html="text"></div>
    <div v-transfer-dom>
      <actionsheet :menus="menus" v-model="showMenus" @on-click-menu="clickMenu"></actionsheet>
    </div>
  </div>
</template>

<script>
  import { XHeader, Actionsheet, TransferDom } from 'vux';
  import Store from 'common/js/store';
  import bookInfo from 'api/bookInfo';

  export default {
    directives: {
      TransferDom
    },
    components: {
      XHeader,
      Actionsheet
    },
    data () {
      return {
        title: '载入中...',
        menus: {
          chapters: '目录',
          sources: '换源'
        },
        text: '',
        showMenus: false
      };
    },
    mounted () {
      this.$emit('hideTab');
      let bookData = this.$route.params.bookData;
      if (bookData) {
        this.setSource(bookData);
      }
    },
    methods: {
      setSource (bookData) {
        let book = window.localStorage.getItem(bookData['_id']);
        if (!book) {
          bookInfo.sources(bookData['_id']).then((data) => {
            if (data.code === 1) {
              let idata = null;
              if (data.data.length > 1) {
                idata = data.data[1];
                return bookInfo.chapters(idata['_id']);
              }
            }
            return null;
          }).then((data) => {
            if (data && data.code === 1) {
              Store.saveSource(data.data['book'], data.data);
              Store.setReadIndex(data.data['book'], 0);
              this.setText(data.data);
            }
          });
        } else {
          book = JSON.parse(book);
          bookInfo.chapters(book['_id']).then((data) => {
            if (data && data.code === 1) {
              Store.saveSource(data.data['book'], data.data);
              this.setText(data.data);
            }
          });
        }
      },
      setText (data) {
        let index = Store.getReadIndex(data['book']);
        if (data.chapters) {
          let info = data.chapters[index];
          bookInfo.getContent(info.link).then((data) => {
            if (data.code === 1) {
              this.title = data.data.chapter.title;
              this.text = data.data.chapter.body.replace(/\n/g, '<br>');
            }
          });
        }
      },
      clickMenu (key) {
        if (key === 'chapters') {
          this.$router.push({
            name: 'chapters',
            params: { bookData: this.$route.params.bookData }
          });
        } else if (key === 'sources') {
          this.$router.push({
            name: 'sources',
            params: { bookData: this.$route.params.bookData }
          });
        }
      },
      clickBack () {
        this.$router.push({
          name: 'bookshelf'
        });
      }
    },
    watch: {
      $route (param) {
        let bookData = this.$route.params.bookData;
        if (bookData) {
          this.setSource(bookData);
        } else {
          this.title = '载入中';
          this.text = null;
        }
      }
    }
  };
</script>

<style>
.content{
  padding: 5px;
  background-color: #C7EDCC;
  line-height: 30px;
  font-size: 18px;
}
</style>
