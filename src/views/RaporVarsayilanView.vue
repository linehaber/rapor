<template>
  <div>
    <form>
      <label for="start-date">Start Date:</label>
      <input type="date" id="start-date" v-model="startDate" @input="getAuthorsWithFilter">
      <label for="end-date">End Date:</label>
      <input type="date" id="end-date" v-model="endDate" @input="getAuthorsWithFilter">
    </form>
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
  name: 'RaporVarsayilan',
  data() {
    return {
      authors: [],
      startDate: null,
      endDate: null,
    };
  },
  methods: {
    getAuthorsWithFilter() {
      this.authors = []
      let start = this.startDate ? this.startDate + 'T00:00:00' : ''
      let end = this.endDate ? this.endDate + 'T23:59:59' : ''
      axios.get(`//localhost:10004/wp-json/wp/v2/users?per_page=100&before=${end}&after=${start}`)
        .then(response => {
          response.data.forEach(author => {
            axios.get(`//localhost:10004/wp-json/wp/v2/posts?author=${author.id}&before=${end}&after=${start}`)
              .then(res => {
                author.post_count = res.headers['x-wp-total']
                this.authors.push(author);
              });
          });
        });
    },
  },
  mounted() {
    this.startDate = new Date().toISOString().slice(0, 10)
    this.endDate = new Date().toISOString().slice(0, 10)
    this.getAuthorsWithFilter();
  },
};
</script>


