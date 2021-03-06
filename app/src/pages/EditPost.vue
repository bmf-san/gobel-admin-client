<template>
  <div class="editpost">
    <h1>EditPost</h1>
    <Loader v-show="loading" />
    <div>
      <Error :error="error" />
      <form @submit.prevent="save">
        <input type="text" name="title" v-model="title" />
        <multiselect
          v-model="tags"
          label="name"
          track-by="id"
          :options="tagOptions"
          :multiple="true"
          :taggable="true"
        ></multiselect>
        <select v-model="categoryId">
          <option disabled value="">Select a category</option>
          <option
            v-for="category in categories"
            :value="category.id"
            :key="category.id"
          >
            {{ category.name }}
          </option>
        </select>
        <textarea v-model="markdown"></textarea>
        <select v-model="status">
          <option disabled value="">Select a status</option>
          <option v-for="status of statuses" :key="status">
            {{ status }}
          </option>
        </select>
        <input type="submit" value="Save" />
      </form>
      <div class="preview" v-html="compileMarkdown"></div>
    </div>
  </div>
</template>

<script>
import Loader from "@/components/Loader.vue";
import Error from "../components/Error";
import apiClient from "../modules/apiClient";
import consts from "../consts/posts";
import marked from "marked";
import Multiselect from "vue-multiselect";
import hljs from "highlight.js";
import "highlight.js/styles/github.css";

export default {
  name: "EditPost",
  components: {
    Loader,
    Error,
    Multiselect
  },
  data() {
    return {
      loading: false,
      error: "",
      title: "",
      tags: [],
      categoryId: "",
      tagOptions: [],
      categories: [],
      markdown: "",
      statuses: consts.STATUSES,
      status: ""
    };
  },
  created() {
    const id = this.$route.params.id;
    this.getPost(id);
    this.getTags();
    this.getCategories();
    marked.setOptions({
      langPrefix: "",
      highlight: function(code, lang) {
        return hljs.highlightAuto(code, [lang]).value;
      }
    });
  },
  computed: {
    compileMarkdown: function() {
      return marked(this.markdown);
    }
  },
  methods: {
    async getPost(id) {
      try {
        this.loading = true;
        await apiClient
          .get(`/private/posts/${id}`, {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("access_token")
            }
          })
          .then(res => {
            this.categoryId = res.data.category.id;
            this.tags = res.data.tags;
            this.title = res.data.title;
            this.markdown = res.data.md_body;
            this.status = res.data.status;

            this.loading = false;
          });
      } catch (e) {
        console.log(e);
      } finally {
        this.loading = false;
      }
    },
    async getTags() {
      try {
        this.loading = true;
        await apiClient
          .get(`/private/tags?page=1&limit=9999`, {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("access_token")
            }
          })
          .then(res => {
            this.tagOptions = res.data;
            this.loading = false;
          });
      } catch (e) {
        console.log(e);
      } finally {
        this.loading = false;
      }
    },
    async getCategories() {
      try {
        this.loading = true;
        await apiClient
          .get(`/private/categories?page=1&limit=9999`, {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("access_token")
            }
          })
          .then(res => {
            this.categories = res.data;
            this.loading = false;
          });
      } catch (e) {
        console.log(e);
      } finally {
        this.loading = false;
      }
    },
    async save() {
      try {
        this.loading = true;
        const tagIds = this.tags.map(obj => {
          return {
            id: obj.id
          };
        });
        await apiClient
          .patch(
            `/private/posts/${this.$route.params.id}`,
            {
              category_id: this.categoryId,
              tags: tagIds,
              title: this.title,
              md_body: this.markdown,
              html_body: this.compileMarkdown,
              status: this.status
            },
            {
              headers: {
                Authorization: "Bearer " + localStorage.getItem("access_token")
              }
            }
          )
          .then(() => {
            this.loading = false;
          });
      } catch (e) {
        this.error = e.response.data.message;
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped></style>
