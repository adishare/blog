<template>
  <section class="articlecard">
    <div id="maincontainer">
  
      <a class="card-link" @click="$router.push({name: 'articledetail',params: {id: article._id}})">
        <article class="blog-card">
          <!-- <img class="post-image" :src="image" /> -->
          <img class="post-image" src="https://picsum.photos/300/300/?random" />
          <div class="article-details">
            <h4 class="post-category"> Category </h4>
  
            <VueEditForm v-if="editMode" :schema='schemaEdit' :model='model'></VueEditForm>
            <div class="buttonbar">
              <button v-if="editMode" type="button" v-on:click="editArticle()" class="btn btn-primary">Update</button>
              <button v-if="editMode" type="button" v-on:click="editMode=false" class="btn btn-secondary">Cancel</button>
            </div>
            <div v-if="!editMode">
              <h3 class="post-title">{{ article.title }}</h3>
              <p class="post-description" v-if="!detail">{{ article.content.slice(0,200) }} <a style="color:red;">... readmore  </a> </p>
              <p class="post-description" v-if="detail">{{ article.content }}</p>
              <p class="post-author">By {{ article.author.firstName+' '+article.author.lastName }}</p>
            </div>
          </div>
  
        </article>
      </a>
  
      <div class="cardfooter">
        <sharecomponent id="sharecmpnt" v-if="user" :article='article' :user='user'></sharecomponent>
        <div id="cmtbtn" @click="$router.push({name: 'articledetail',params: {id: article._id}}); goComment()"> Comment ( {{comments.length}} ) </div>
        <div v-if="article.author._id == user._id & detail" @click="editMode = true"> Edit </div>
        <div v-if="article.author._id == user._id & detail" @click="deleteArticle()"> Delete </div>
      </div>
  
      <div v-if="detail" class="commentbar">
        <div class='articlecomments'>
          <div class="usercomment" v-for="comment in comments" :key="comment._id">
            <h6> " {{ comment.comment }} " </h6>
            <h5> - {{ comment.user.firstName+' '+comment.user.lastName }} <br> <i> at {{ formatthis(comment.createdAt) }} </i></h5>
            <h5 v-if="comment.user._id == user._id"> <i> <button type="button" @click="deleteComment(comment._id)" class="btn">REMOVE</button> </i> </h5>
          </div>
        </div>
        <div v-if="user" class="commentarea">
          <VueForm :schema='schema' :model='model'></VueForm>
        </div>
        <div v-if="user" style="align-self:start;">
          <button type="button" @click="postComment()" class="btn btn-primary">Post Comment</button>
        </div>
      </div>
  
    </div>
  </section>
</template>

<script>
  import VueForm from "vue-form-generator";
  import axios from 'axios';
  import $ from 'jquery';
  import moment from 'moment';
  import sharecomponent from './Share.vue'
  
  export default {
    name: 'articlecard',
    props: ['article', 'articles', 'detail', 'user', 'getArticles'],
    components: {
      VueForm: VueForm.component,
      sharecomponent,
      VueEditForm: VueForm.component
    },
    created() {
      this.getComments()
    },
    data() {
      return {
        editMode: false,
        comments: [],
  
        model: {
          comment: '',
  
          titleEdit: this.article.title,
          contentEdit: this.article.content
        },
  
        schema: {
          fields: [{
            type: 'textArea',
            model: 'comment',
            placeholder: 'Your Comment ...',
            rows: 3,
            max: 1000,
            hint: 'max 1000 char',
            validator: VueForm.validators.string
          }]
        },
  
        schemaEdit: {
          fields: [{
              type: "input",
              inputType: "text",
              label: 'Title Edit',
              model: "titleEdit",
              placeholder: "Article Title",
              required: true,
            },
            {
              type: "textArea",
              model: "contentEdit",
              label: 'Content Edit',
              placeholder: "Content",
              required: true,
              rows: 15,
              validator: VueForm.validators.string
            }
          ]
        }
      }
    },
    methods: {
      deleteComment(commentId) {
        axios.delete(`http://localhost:3000/comments/${commentId}`)
          .then(() => {
            this.getComments()
          }).catch((err) => {
            console.log(err);
          });
      },
  
      editArticle() {
        let self = this
        axios.put(`http://localhost:3000/articles/${this.article._id}`, {
            title: self.model.titleEdit,
            content: self.model.contentEdit,
          })
          .then(() => {
            this.getArticles()
            this.$router.push(`/`)
          }).catch((err) => {
            console.log(err)
          });
      },
  
      deleteArticle() {
        axios.delete(`http://localhost:3000/articles/${this.article._id}`)
          .then(() => {
            this.getArticles()
            this.article = this.article
            this.$router.push('/')
          }).catch((err) => {
            console.log(err);
          });
      },
      postComment() {
        axios.post(`http://localhost:3000/comments/${this.article._id}`, {
            user: this.user._id,
            comment: this.model.comment
          })
          .then(() => {
            this.getComments()
            this.model.comment = ''
          }).catch((err) => {
            console.log(err);
          });
      },
      getComments() {
        axios.get(`http://localhost:3000/comments/${this.article._id}`)
          .then((result) => {
            this.comments = result.data
          }).catch((err) => {
            console.log(err);
          });
      },
      formatthis(date) {
        return moment(date).format('DD MMM YY - hh:mm')
      },
      goComment() {
        $('#comment').focus();
      }
  
    },
    watch: {
      '$route.params.id': function(newVal) {
        this.param = newVal
        this.article = this.articles.filter(item => {
          return item._id == this.param
        })[0]
        this.getComments()
      },
  
      // articles: {
      //   handler: function(newValue) {
      //     this.article = newValue.filter(item => {
      //       return item._id == this.$route.params.id
      //     })[0]
      //   },
      //   deep: true,
      // }
  
    },
  }
</script>

<style scoped lang="css">
  .commentbar {
    /* width: 100%; */
    display: grid;
    grid-template-rows: 1fr minmax(30px, 100px) minmax(20px, 50px);
    justify-items: center;
    align-items: center;
    background-color: #fff;
    box-shadow: 0 0.1875rem 1.5rem rgba(0, 0, 0, 0.2);
    padding-top: 15px;
  }
  
  .buttonbar {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 20px;
  }
  
  .articlecomments {
    width: 100%;
    max-width: 90%;
    padding-top: 20px;
  }
  
  .commentarea {
    width: 100%;
    max-width: 90%;
    padding-top: 20px;
  }
  
  .articlecard {
    transition: .5s;
  }
  
  * {
    box-sizing: border-box;
  }
  
  *::before,
  *::after {
    box-sizing: border-box;
  }
  
  #maincontainer {
    max-width: 90%;
    margin: 0 auto;
    text-align: left;
  }
  
  #sharecmpnt {
    grid-column: 1 / span 2;
  }
  
  .cardfooter {
    grid-column: 1 / span 2;
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    justify-items: center;
    align-items: center;
    background-color: #fff;
    box-shadow: 0 0.1875rem 1.5rem rgba(0, 0, 0, 0.2);
    min-height: 40px;
    /* z-index: 10; */
  }
  
  .cardfooter div {
    width: 100%;
    height: 100%;
    display: grid;
    justify-items: center;
    align-items: center;
  }
  
  i {
    font-size: 13px;
  }
  
  h6 {
    text-align: center;
  }
  
  h5 {
    text-align: right;
  }
  
  .cardfooter div:hover {
    cursor: pointer;
    background-color: rgb(219, 219, 219);
  }
  
  .blog-card {
    display: flex;
    flex-direction: row;
    background: #fff;
    box-shadow: 0 -0.2875rem 1.5rem rgba(0, 0, 0, 0.2);
    border-radius: 0.375rem;
    overflow: hidden;
    transition: .1s;
  }
  
  .blog-card:hover {
    cursor: pointer;
    box-shadow: 0 -0.2875rem 1.1rem rgba(0, 0, 0, 0.3);
  }
  
  .card-link {
    position: relative;
    display: block;
    color: inherit;
    text-decoration: none;
  }
  
  .card-link:hover .post-title {
    transition: color 0.3s ease;
    color: #e04f62;
  }
  
  .card-link:hover .post-image {
    transition: opacity 0.3s ease;
    opacity: 0.9;
  }
  
  .post-image {
    transition: opacity 0.3s ease;
    display: block;
    width: 100%;
    object-fit: cover;
  }
  
  .article-details {
    padding: 1.5rem;
  }
  
  .post-category {
    display: inline-block;
    text-transform: uppercase;
    font-size: 0.75rem;
    font-weight: 700;
    line-height: 1;
    letter-spacing: 0.0625rem;
    margin: 0 0 0.75rem 0;
    padding: 0 0 0.25rem 0;
    border-bottom: 0.125rem solid #ebebeb;
  }
  
  .post-title {
    transition: color 0.3s ease;
    font-size: 1.125rem;
    line-height: 1.4;
    color: #121212;
    font-weight: 700;
    margin: 0 0 0.5rem 0;
  }
  
  .post-author {
    font-size: 0.875rem;
    line-height: 1;
    margin: 1.125rem 0 0 0;
    padding: 1.125rem 0 0 0;
    border-top: 0.0625rem solid #ebebeb;
  }
  
  @media (max-width: 40rem) {
    .blog-card {
      flex-wrap: wrap;
    }
  }
  
  @supports (display: grid) {
    .post-image {
      height: 100%;
    }
    .blog-card {
      display: grid;
      grid-template-columns: 1fr 2fr;
      grid-template-rows: 1fr;
    }
    @media (max-width: 40rem) {
      .blog-card {
        grid-template-columns: auto;
        grid-template-rows: 12rem 1fr;
      }
    }
  }
</style>
