<template>
  <div>
    <table>
      <tr>
        <th>Author Name</th>
        <th>Post Count</th>
      </tr>
      <tr v-for="author in authors" :key="author.id">
        <td>{{ author.name }}</td>
        <td>{{ author.post_count }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Rapor',
  data() {
    return {
      authors: [],
    };
  },
  methods: {
    getAuthors(page = 1) {
      axios.get(`//localhost:10004/wp-json/wp/v2/users?per_page=100&page=${page}`)
        .then(response => {
          response.data.forEach(author => {
            axios.get(`//localhost:10004/wp-json/wp/v2/posts?author=${author.id}`)
            .then(res => {
                author.post_count = res.headers['x-wp-total']
                this.authors.push(author);
            });
          });
          if (response.headers.link && /page=(\d+)/.exec(response.headers.link)) {
              let nextPage = /page=(\d+)/.exec(response.headers.link)[1];
              this.getAuthors(nextPage);
          }
          else {
            // Do something if response headers do not contain the link
            console.log("Link header not found in the response")
          }
        });
    },
  },
  mounted() {
    this.getAuthors();
  },
};
</script>

