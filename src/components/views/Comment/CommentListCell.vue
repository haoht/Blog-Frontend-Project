<template>
  <div class="comment-list-cell">
    <iv-row>
      <iv-col :xs="cellSpan('xs')" :sm="cellSpan('sm')" :md="cellSpan('md')" :lg="cellSpan('lg')" :xl="cellSpan('xl')">
        <div class="comment-main">
          <iv-row :gutter="8">
            <iv-col :xs="cellLeftSpan('xs')" :sm="cellLeftSpan('sm')" :md="cellLeftSpan('md')"
                    :lg="cellLeftSpan('lg')" :xl="cellLeftSpan('xl')">
              <div class="avatar">
                <img :src="postImageBaseUrl + '/comment/avatar/' + avatarImage(comment.author)" alt="">
              </div>
            </iv-col>
            <iv-col :xs="cellRightSpan('xs')" :sm="cellRightSpan('sm')" :md="cellRightSpan('md')"
                    :lg="cellRightSpan('lg')" :xl="cellRightSpan('xl')">
              <div class="content">
                <p class="title">
                  <span class="name" :class="theme" v-if="comment.author !== null"><a>{{ comment.author.nick_name }}</a></span>
                  <span class="name-tag">Mod</span>
                  <span class="reply-icon" :class="theme" v-if="comment.reply_to_author !== null"><iv-icon
                    type="forward"></iv-icon></span>
                  <span class="reply-name" :class="theme"
                        v-if="comment.reply_to_author !== null"><a>{{ comment.reply_to_author.nick_name }}</a></span>
                  <span class="time">{{ comment.add_time | socialDate }}</span>
                </p>
                <p class="comment-main-content" :class="theme" v-if="comment.detail" v-html="comment.detail.formatted_content"
                   ref="content" v-viewer="{movable: false}"></p>
                <div class="operate-area" :class="theme">
                  <span class="like" @click="likeComment(comment)"><iv-icon type="thumbsup"></iv-icon> {{ comment.like_num }}</span>
                  <span class="unlike" @click="unlikeComment(comment)"><iv-icon type="thumbsdown"></iv-icon> {{ comment.unlike_num }}</span>
                  <span class="reply"><a @click="showEditor = !showEditor"><iv-icon
                    type="forward"></iv-icon> 回复</a></span>
                  <!--<iv-dropdown>-->
                  <!--<span class="iv-dropdown-link">-->
                  <!--<iv-icon type="android-share-alt"></iv-icon> 分享 <iv-icon type="arrow-down-b"></iv-icon>-->
                  <!--</span>-->
                  <!--<iv-dropdown-menu slot="list">-->
                  <!--<iv-dropdown-item>菜单</iv-dropdown-item>-->
                  <!--<iv-dropdown-item>菜单</iv-dropdown-item>-->
                  <!--<iv-dropdown-item>菜单</iv-dropdown-item>-->
                  <!--<iv-dropdown-item disabled>菜单</iv-dropdown-item>-->
                  <!--<iv-dropdown-item divided>菜单</iv-dropdown-item>-->
                  <!--</iv-dropdown-menu>-->
                  <!--</iv-dropdown>-->
                  <!--<span class="reply"><a>查看评论列表</a></span>-->
                </div>
                <div class="comment-area" v-show="showEditor">
                  <div class="reply-editor" :class="{spread: spreadEditor}">
                    <mavon-editor :post="post"
                                  :replyToComment="comment"
                                  :theme="theme"
                                  @valueChanged="valueChanged"
                                  @publishedComment="publishedComment"></mavon-editor>
                  </div>
                </div>
              </div>
            </iv-col>
          </iv-row>
        </div>
      </iv-col>
    </iv-row>
  </div>
</template>

<script type="text/ecmascript-6">
  import MavonEditor from '@/components/views/MavonEditor';
  // highlight.js引入
  import hljs from '@/common/js/highlight.pack';
  // Api
  import {likeOrUnlikeComment} from '@/api/api';
  // utils
  import {hexMd5} from '@/common/js/md5';

  var HLJS = hljs;

  const CELL_LEFT_SPAN = {
    'xs': 3,
    'sm': 3,
    'md': 2,
    'lg': 2
  };
  const CELL_RIGHT_SPAN = {
    'xs': 24 - CELL_LEFT_SPAN['xs'],
    'sm': 24 - CELL_LEFT_SPAN['sm'],
    'md': 24 - CELL_LEFT_SPAN['md'],
    'lg': 24 - CELL_LEFT_SPAN['lg']
  };

  export default {
    props: {
      post: {
        Type: Object,
        default: undefined
      },
      comment: {
        Type: Object,
        default: undefined
      },
      theme: {
        Type: String,
        default: ''
      }
    },
    data() {
      return {
        showEditor: false,
        spreadEditor: false
      };
    },
    methods: {
      avatarImage(author) {
        // 随机固定的头像图片名
        let idStr = author.id + '';
        let start = author.nick_name.length + idStr.length > 31 ? 31 : author.nick_name.length + idStr.length;
        return hexMd5(author.nick_name + idStr).slice(start, start + 1) + '.png';
      },
      addCodeLineNumber() {
        // 添加行号
        if (this.$refs.content) {
          let blocks = this.$refs.content.querySelectorAll('pre code');
          blocks.forEach((block) => {
            HLJS.highlightBlock(block);
            // 去前后空格并添加行号
            let reg = /<ul(.*?)><li(.*?)>[\s\S]*?<\/li><\/ul>/gm;
            if (reg.test(block.innerHTML)) return;
            block.innerHTML = '<ul><li>' + block.innerHTML.replace(/(^\s*)|(\s*$)/g, '').replace(/\n/g, '\n</li><li>') + '\n</li></ul>';
          });
        }
      },
      cellSpan(size) {
        var span = {};
        span['offset'] = CELL_LEFT_SPAN[size] * parseInt(this.comment.comment_level);
        span['span'] = 24 - span['offset'];
        return span;
      },
      cellLeftSpan(size) {
        var span = {};
        span['span'] = CELL_LEFT_SPAN[size];
        return span;
      },
      cellRightSpan(size) {
        var span = {};
        span['span'] = CELL_RIGHT_SPAN[size];
        return span;
      },
      valueChanged(flag) {
        this.spreadEditor = flag;
      },
      publishedComment(comment) {
        console.log(comment);
        this.$emit('publishedComment', comment);
      },
      likeComment(comment) {
        likeOrUnlikeComment({
          comment_id: comment.id,
          operation: true
        }).then((response) => {
          comment.like_num += 1;
          this.$Message.success('点赞成功');
        }).catch((error) => {
          console.log(error);
        });
      },
      unlikeComment(comment) {
        likeOrUnlikeComment({
          comment_id: comment.id,
          operation: false
        }).then((response) => {
          comment.unlike_num += 1;
          this.$Message.success('吐槽成功');
        }).catch((error) => {
          console.log(error);
        });
      }
    },
    mounted() {
      this.$nextTick(() => {
        this.addCodeLineNumber();
        // 添加图片前缀
        if (this.$refs.content) {
          this.resolveImageUrl(this.$refs.content.querySelectorAll('img'));
        }
      });
    },
    components: {
      'mavon-editor': MavonEditor
    }
  };
</script>

<style lang="stylus" type="text/stylus" rel="stylesheet/stylus">
  @import "../../../common/stylus/theme.styl";
  @import "../.././../common/stylus/comment.styl";

  .comment-list-cell
    position relative
    text-align left
    .avatar
      text-align center
      img
        border-radius $border-radius
        width 100%
    .content
      margin 5px 0 20px
      .title
        font-size 0
        margin-bottom 5px
        line-height 18px
        .name
          a
            font-size 15px
            color $color-main-primary
            font-weight 700
            &:hover
              text-decoration underline
          &.dark-theme
            a
              color $color-secondary-warning
        .name-tag
          font-size 12px
          background-color $color-secondary-info
          padding 2px 5px
          margin 0 5px
          color #fff
          border-radius $border-radius
        .reply-icon
          font-size 15px
          color $light
        .reply-name
          font-size 15px
          margin 0 5px
          a
            color $dark
            &:hover
              color $color-main-primary
              text-decoration underline
          &.dark-theme
            a
              color $color-gradually-gray-71
              &:hover
                color $color-secondary-warning
                text-decoration underline
        .time
          font-size 13px
          color $light
          margin-left 8px
      .comment-main-content
        font-size 16px
        line-height 24px
        margin 10px 0 15px
        &.dark-theme
          color $color-gradually-gray-71
    .operate-area
      margin-top 8px
      font-size 14px
      span
        margin-right 10px
      .iv-dropdown-link
        cursor pointer
      .like, .unlike
        color $light
        font-weight 300
        cursor pointer
      .reply
        cursor pointer
      &.dark-theme
        .iv-dropdown-link
          &:hover
            color $color-secondary-warning
        .reply
          a
            color $color-secondary-warning
    .comment-area
      margin-bottom 10px
      .reply-editor
        margin-top 15px
        height 200px
        transition height 0.7s
        &.spread
          height 300px
      p.comment-tip
        a
          font-size 14px
          &:hover
            color $color-main-primary
</style>
