<template>
  <div>
    <h1 class="my-3 center">
      <strong>{{ deviceName }}</strong>
    </h1>

    <h4>
      {{ deviceStatus }} -
      <a
        :href="'https://maps.google.com/?q=' + deviceLocation"
        target="_blank"
        >{{ deviceLocation }}</a
      >
    </h4>

    <div class="d-flex mt-5">
      <div class="date-picker">
        <h5>Date Picker</h5>

        <v-date-picker
          v-model="range"
          mode="dateTime"
          :masks="masks"
          is-range
        />
      </div>

      <div class="measurements-table">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">Date</th>
              <th scope="col">Temperature</th>
              <th scope="col">Humidity</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(m, idx) in displayedDevices" :key="idx">
              <th scope="row">
                {{ m.date }}
              </th>
              <td>{{ m.temperature }}</td>
              <td>{{ m.humidity }}</td>
            </tr>
          </tbody>
        </table>

        <nav>
          <p style="display: inline">Page {{ page }} of {{ pages.length }}</p>

          <ul class="pagination">
            <li class="page-item">
              <button
                type="button"
                class="page-link d-inline"
                v-if="page > 1"
                @click="page--"
              >
                Previous
              </button>
            </li>
            <li class="page-item">
              <button
                type="button"
                @click="page++"
                v-if="page < pages.length"
                class="page-link d-inline"
              >
                Next
              </button>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  </div>
</template>






<script>
var dateFormat = require("dateformat");
export default {
  name: "Measurements",
  data() {
    return {
      posts: [""],
      page: 1,
      deviceLocation: "Loading location...",
      deviceStatus: "Loading status...",
      perPage: 9,
      pages: [],
      timer: "",
      range: {
        start: new Date(),
        end: new Date(),
      },
      masks: {
        input: "DD-MM-YYYY hh:mm",
      },
    };
  },

  methods: {
    getPosts() {
      fetch(
        "http://weatherstation.tk:5000/dso/measurements/" +
          this.$route.params.id +
          "?start=" +
          dateFormat(this.range.start, "dd-mm-yyyy HH:MM") +
          "&end=" +
          dateFormat(this.range.end, "dd-mm-yyyy HH:MM")
      )
        .then((response) => response.json())
        .then((json) => {
          this.posts = json.data;
        });
    },
    getDeviceInfo() {
      fetch(
        "http://weatherstation.tk:5000/dso/devices/" + this.$route.params.id
      )
        .then((response) => response.json())
        .then((json) => {
          this.deviceLocation = json.data[0].location;
          this.deviceStatus = json.data[0].status;
        });
    },
    setPages() {
      let numberOfPages = Math.ceil(this.posts.length / this.perPage);
      this.pages = [];
      for (let index = 1; index <= numberOfPages; index++) {
        this.pages.push(index);
      }
    },
    paginate(posts) {
      let page = this.page;
      let perPage = this.perPage;
      let from = page * perPage - perPage;
      let to = page * perPage;
      return posts.slice(from, to);
    },
  },
  computed: {
    displayedDevices() {
      return this.paginate(this.posts);
    },
    deviceName() {
      return this.$route.params.id;
    },
  },
  watch: {
    posts() {
      this.setPages();
    },
  },
  created() {
    this.getPosts();
    this.getDeviceInfo();
    this.timer = setInterval(this.getPosts, 2000);
    this.timer = setInterval(this.getDeviceInfo, 2000);
  },
  filters: {
    trimWords(value) {
      return value.split(" ").splice(0, 20).join(" ") + "...";
    },
  },
};
</script>

<style>
.measurements-table {
  flex: 1;
}

.date-picker {
  flex: 0;
  margin-top: 10px;
  margin-right: 30px;
}
</style>