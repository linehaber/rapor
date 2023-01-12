<template>
  <div>
    <table>
      <tr>
        <th>Author Name</th>
        <th>Post Count</th>
      </tr>
      <tr v-for="author in authors" :key="author.id">
        <td>{{ author.name }}</td>
        <td>{{ author.postCount }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'About',
  data() {
    return {
      posts: null,
      authors: [], // empty array
    };
  },
  mounted() {
    axios.get('//localhost:10004/wp-json/wp/v2/posts/?per_page=100')
      .then(response => {
        this.posts = response.data;
        this.getAuthors();
      });
  },
  methods: {
    getAuthors() {
      this.authors = []
      let authorIds = this.posts.map(post => post.author);
      authorIds = [...new Set(authorIds)]; // remove duplicates
      authorIds.forEach(authorId => {
        axios.get(`//localhost:10004/wp-json/wp/v2/users/${authorId}`)
          .then(response => {
            let author = {
              id: authorId,
              name: response.data.name,
              postCount: 0,
            };
            this.posts.forEach(post => {
              if (post.author === authorId) {
                author.postCount++;
              }
            });
            this.authors.push(author);
          });
      });
    }
  }
};
</script>


<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
