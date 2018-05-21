<template>
<div class="container">
  <h3 class="text-center">Vue新闻列表</h3>
		<section class="callout secondary">
			<h5 class="text-center">请选择新闻分类</h5>
			<form>
				<div class="row">
					<div class="large-6 columns">
						<select v-model="section">
							<option v-for="section in sections" :value="section">{{ section }}</option>
						</select>
					</div>
					<div class="medium-6 columns">
						<a @click="getPosts(section)" class="button expanded">确认选择</a>
					</div>
				</div>
			</form>
		</section>

		<h5 class="text-center uppercase" v-if="!loading">{{ title }}</b></h5>
		<div v-if="loading" class="loader"><img src="//cdnjs.cloudflare.com/ajax/libs/semantic-ui/0.16.1/images/loader-large.gif" alt="loader"></div>

		<news-list v-if="!loading" :results="results"></news-list>
</div>
</template>

<script>
import Vue from 'vue';
import axios from 'axios';
const NYTBaseUrl = "https://api.nytimes.com/svc/topstories/v2/";
const ApiKey = 'f860e8ae9e8b478c872ad8acc89075e2';
const SECTIONS = "home, arts, automobiles, books, business, fashion, food, health, insider, magazine, movies, national, nyregion, obituaries, opinion, politics, realestate, science, sports, sundayreview, technology, theater, tmagazine, travel, upshot, world"; // From NYTimes
const buildUrl = function (url) {
    return NYTBaseUrl + url + ".json?api-key=" + ApiKey;
}

export default {
  name: 'index',
  data () {
    return {
      results: [],
      sections: SECTIONS.split(', '), // 生成分类数组
      section: 'home', // 默认分类 home
      loading: true,
      title: ''
    }
  },
  mounted () {
    this.getPosts('home');
  },
  methods: {
    getPosts(section) {
      let url = buildUrl(section);
      axios.get(url).then((response) => {
        this.loading = false;
        this.results = response.data.results;
        let title = this.section !== 'home' ? "今日【"+ this.section + "】新闻" : "今日头条新闻";
        this.title = title + "(" + response.data.num_results+ ")";
      }).catch((error) => { console.log(error); });
    }
  }
}

Vue.component('news-list', {
  props: ['results'],
  template: `
    <section>
      <div class="row" v-for="posts in processedPosts">
        <div class="columns large-3 medium-6" v-for="post in posts">
          <div class="card">
          <div class="card-divider">
          {{ post.title }}
          </div>
          <a :href="post.url" target="_blank"><img :src="post.image_url"></a>
          <div class="card-section">
            <p>{{ post.abstract }}</p>
          </div>
        </div>
        </div>
      </div>
  </section>
  `,
  computed: {
    processedPosts() {
      let posts = this.results;
      // 添加新闻图片路径
      posts.map(post => {
        let imgObj = post.multimedia.find(media => media.format === "superJumbo");
        post.image_url = imgObj ? imgObj.url : "//placehold.it/300x200?text=N/A";
      });
      // 数组调整分组
      let i, j, chunkedArray = [], chunk = 4;
      for (i=0, j=0; i < posts.length; i += chunk, j++) {
        chunkedArray[j] = posts.slice(i,i+chunk);
      }
      return chunkedArray;
    }
  }
})

</script>

<style scoped>
/* 引入样式 */
@import '//cdnjs.cloudflare.com/ajax/libs/foundation/6.3.1/css/foundation.min.css';
.container {
  padding: 20px 0
}
.text-center{
  margin-bottom:20px;
}
.column:last-child:not(:first-child), .columns:last-child:not(:first-child) {
  float: left;
}
.main_heading {
  margin: 10px 0 30px;
}
.callout {
  margin-bottom: 40px;
}
select {
  text-transform:capitalize;
}
.loader {
  text-align: center;
  padding: 40px;
}
.uppercase {
  text-transform: uppercase;
}
</style>
