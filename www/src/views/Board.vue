<template>
  <div class="board">
    <div class="page-loading loader-inner ball-pulse" v-if="isLoading">
      <div> </div> <div> </div> <div> </div>
    </div>
    <div class="container" v-show="! isLoading">
      <div class="board-description">
        <div class="media">
          <content class="media-content">
            <h4 class="head"> {{ description }}</h4>
          </content>
        </div>
      </div>
      <div class="top-posts-section">
        <div class="top-post" v-for="(post, index) in topPosts" :key="post.id" v-if="! index || showAllTopPosts">
          <span class="tag is-warning" :class="index ? 'invisible' : 'visible'">置顶</span>
          <router-link target="_blank" :to="getArticle(post.gid)">
            <p> <strong> {{ post.title }}</strong> </p>
          </router-link>
          <span @click="showAllTopPosts = showAllTopPosts ? false : true" v-if="! index" class="more">
            <i class="icon iconfont" :class="showAllTopPosts ? 'icon-less' : 'icon-moreunfold'"></i>
          </span>
        </div>
      </div>
      <div class="posts">
        <div class="post" v-for="(article, index) in posts" :key="index">
          <router-link target="_blank" :to="getArticle(article.gid)">
            <div class="poster media">
              <div class="media-content">
                <div class="content">
                  <p> <strong> {{ article.title }}</strong> </p>
                </div>
                <nav class="article-footer level is-mobile">
                  <div class="level-left">
                    <span class="level-item">{{ article.poster }}</span>
                    <span class="level-item"> {{ article.replyTime }} 更新</span>
                  </div>
                  <div class="level-right">
                    <span class="level-item"> {{ article.replyCount }} 人回复</span>
                  </div>
                </nav>
              </div>
            </div>
          </router-link>
          <hr>
        </div>
      </div>
    </div>
    <section class="paginate" v-show="! isLoading && totalPage > 1">
      <div class="card">
        <header class="columns is-mobile paginate-items">
          <div class="column">
            <a @click="getPrevPage">上一页</a>
          </div>
          <div class="column">
            <a>{{ currentPage + '/' + totalPage }}</a>
          </div>
          <div class="column">
            <a @click="getNextPage">下一页</a>
          </div>
        </header>
      </div>
    </section>

    <float-button @click.native="newPost()"></float-button>

  </div>
</template>

<script>
import * as api from 'api/board'
import FloatButton from 'components/FloatButton';

export default {
  data () {
    return {
      query: {
        board: null
      },

      isLoading: true,
      showAllTopPosts: false,

      // pagination
      currentPage: 1,
      totalPage: 1,

      canPost: false,
      isAdmin: false,
      isBM: false,
      name: '',
      description: '',

      posts: [],
      topPosts: [],
      cachedPages: {}
    }
  },

  components: {
    FloatButton
  },

  created() {
    this.query = this.$route.params;
    this.fetchBoards();
  },

  methods: {
    fetchBoards() {
      this.isLoading = true;
      const page = this.currentPage;
      if (page in this.cachedPages) {
        this.isLoading = false;
        return this.posts = this.cachedPages[page];
      }

      api.getBoard(this.query.board, { page }).then((res) => {
        if (! res.success) {
          return false;
        }

        const data = res.data;
        this.name = data.name;
        this.description = data.description;
        this.totalPage = data.pagination.total;
        this.canPost = data.canPost;
        this.isAdmin = data.Admin;
        this.isBM = data.isBM;

        this.topPosts = []; this.posts = [];
        for (let i in data.posts) {
          if (data.posts[i].tag == 'top') {
            this.topPosts.push(data.posts[i]);
          } else {
            this.posts.push(data.posts[i]);
          }
        }

        document.body.scrollTop = document.documentElement.scrollTop = 0;
        this.isLoading = false;
      });
    },

    getArticle(gid) {
      return `/article/${this.name}/${gid}`;
    },

    getPrevPage() {
      if (this.currentPage <= 1) {
        return false;
      }
      this.currentPage --;
      this.fetchBoards();
    },

    getNextPage() {
      if (this.currentPage >= this.totalPage) {
        return false;
      }
      this.currentPage ++;
      this.fetchBoards();
    },

    newPost() {
      let url = `/post?type=new&board=${this.query.board}`;
      this.$router.push(url);
    }
  }
}
</script>

<style scoped>
.container {
    padding: 12px 12px;
    background-color: #fff;
}

.board-description {
    padding-bottom: 12px;
}

hr {
    margin: 18px 0 18px 0;
}
.paginate {
    text-align: center;
    margin: 1px;
}
.paginate-items {
    margin-left: 0;
    margin-right: 0;
}
.top-posts-section {
    background-color: whitesmoke;
    margin-bottom: 12px;
}
.top-post {
    padding: 4px 0;
}
.top-post p {
    color: #4a4a4a;
    display: inline-block;
    margin-left: .3rem;
    white-space: nowrap;
    text-overflow: ellipsis;
    max-width: 250px;
    overflow: hidden;
    vertical-align: bottom;
}
.top-post .more {
    float: right;
    color: #ff3860;
}
.visible {
    opacity: 1;
}
.invisible {
    opacity: 0;
}
.article-footer {
    color: #4a4a4a;
}
</style>

