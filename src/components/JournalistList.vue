<template>
  <div class="container">
    <h1>Mastodon Journalists</h1>
    <div v-if="!host" class="d-flex justify-content-center form-check form-switch">
      <input class="form-check-input" type="checkbox" id="flexSwitchCheckChecked" v-model="selectHost">
      <label class="form-check-label mx-2"  for="flexSwitchCheckChecked">Type | Select</label>
    </div>
    <label v-if="!host && selectHost" class="form-label">
      Select Your Mastodon Host:
      <select class="form-select" v-model="mastodonHost">
        <option v-for="(host, index) in hosts" :value="host" :key="'hosts-' + index" v-text="host"></option>
      </select>
      This only appears, if ?host=&lt;your-mastodon-host&gt; isn't in the call url, select from available hosts as your base host for appropriate page redirection
    </label>
    <label v-if="!host && !selectHost" class="form-label">
      Enter Your Mastdon Host:
      <input class="form-control" v-model="mastodonHost">
      This only appears, if ?host=&lt;your-mastodon-host&gt; isn't in the call url, select from available hosts as your base host for appropriate page redirection
    </label>
    <ul class="pagination justify-content-center">
      <li v-for="index in pagedJournalists.length" class="page-item m-0" :key="'nav-' + index">
        <a class="page-link" :class="{'active': page === (index - 1)}" href="#" @click="updatePage(index)"
           v-text="index"></a>
      </li>
    </ul>
    <div class="row gx-2">
      <div v-for="(journalist, index) in readyJournalists"
           class="col-6 col-sm-4 col-lg-2 gy-2" :key="'journalists-' + page + '-' + index">
        <div class="card">
          <img class="card-image" :src="journalistInfo[journalist].url" alt="Profile Image">
          <div class="card-body">
            <h5 class="card-title card-title--size" v-text="journalist"></h5>
            <a :href="createUrl(journalist)" class="btn btn-default">Go</a>
            <!-- Tiny bit risky, but as long as mastodon instances already control what gets put in the summary
             there should be no injection issue, also this is a static website, so...-->
            <details v-if="journalistInfo[journalist].summary">
              <summary>Basic Info</summary>
              <div v-html="journalistInfo[journalist].summary"></div>
            </details>
          </div>
        </div>
      </div>
    </div>
    <ul class="pagination mt-4 justify-content-center">
      <li v-for="index in pagedJournalists.length" class="page-item m-0" :key="'nav-' + index">
        <a class="page-link" :class="{'active': page === (index - 1)}" href="#" @click="updatePage(index)"
           v-text="index"></a>
      </li>
    </ul>
  </div>
</template>

<script>
import {accountsString} from "@/resources/journalists";

const placeholder = require('../assets/placeholder.png');
const journalists = accountsString.split(";").sort();

export default {
  name: 'JournalistList',
  data() {
    return {
      loading: false,
      selectHost: true,
      page: 0,
      pageSize: 50,
      invalidHosts: ['localhost', 'mastodon-journalists.netlify.app'],
      mastodonHost: null,
      journalistInfo: journalists.reduce((acc, journalist) => {
        acc[journalist] = {}
        acc[journalist].loaded = false;
        acc[journalist].url = placeholder;
        acc[journalist].summary = '';
        return acc;
      }, {})
    }
  },
  computed: {
    host(){
      const params = new Proxy(new URLSearchParams(window.location.search), {
        get: (searchParams, prop) => searchParams.get(prop),
      });
      return params.host || null;
    },
    hosts(){
      const hostsList = journalists.map((journalist) => {
        const [, host] = this.getHandleHost(journalist);
        return host;
      }).filter(Boolean);
      hostsList.push(this.host);
      return Array.from(new Set(hostsList)).sort();
    },
    readyJournalists() {
      return this.pagedJournalists[this.page].filter((journalist) => {
        return this.journalistInfo[journalist].loaded;
      })
    },
    pagedJournalists() {
      const processedJournalists = journalists;
      const pagedJournalists = [];
      for (let [i, page] = [0, -1]; i < processedJournalists.length; i++) {
        if (!(i % this.pageSize)) {
          page++;
          pagedJournalists[page] = [];
        }
        pagedJournalists[page].push(processedJournalists[i]);
      }
      return pagedJournalists;
    }
  },
  methods: {
    getHandleHost(journalist){
      return journalist.split("@").filter(Boolean);
    },
    createUrl(journalist) {
      const [targetHandle, targetHost] = this.getHandleHost(journalist);
      const host = this.isValidHost(this.mastodonHost) ? this.mastodonHost : targetHost;
      const handle = this.isValidHost(this.mastodonHost) ? journalist : `@${targetHandle}`;
      return `https://${host}/${handle}`;
    },
    createImageUrl(journalist) {
      const [handle, host] = journalist.split("@").filter(Boolean);
      const jsonUrl = `https://${host}/users/${handle}.json`;
      return fetch(jsonUrl)
          .then(response => response.json())
          .then((json) => {
            this.journalistInfo[journalist].url = json.icon.url;
            this.journalistInfo[journalist].summary = json.summary;
          })
          .catch(() => {
          })
          .then(() => {
            this.journalistInfo[journalist].loaded = true;
          })
    },
    loadJournalistImages() {
      this.pagedJournalists[this.page].forEach((journalist) => {
        if (this.journalistInfo[journalist].url === placeholder) {
          this.createImageUrl(journalist);
        }
      })
    },
    updatePage(index) {
      this.page = (index - 1);
      this.loadJournalistImages();
    },
    isValidHost(host) {
      return host && !this.invalidHosts.includes(host);
    },
  },
  created() {
    this.loadJournalistImages();
    this.mastodonHost = this.host;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.card-image {
  width: 100%;
  height: auto;
}

.card-title--size {
  font-size: 12pt;
}

h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
