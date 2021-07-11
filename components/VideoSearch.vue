<!-- Please remove this file from your project -->
<template>
  <div class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0">
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
      <div class="mt-8 bg-white overflow-hidden shadow sm:rounded-lg p-6">
        <h2 class="text-2xl leading-7 font-semibold text-center border-b mb-5">
          Youtube Видео Поиск
        </h2>
        <div class="flex flex-row justify-center align-center">
          <input v-model="query" type="text" class="mr-2">
          <button @click="search()" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Искать</button>
        </div>
        <div class="m-5">
          {{ items.length > 0 ? 'результат поиска по запросу: '+this.searched : '' }}
        </div>
        <div class="shadow-md mt-5">
          <div
            class="tab w-full overflow-hidden border-t"
            v-for="item in items"
            :key="item.etag"
          >
            <input @click="openPlayer(item)" type="radio" class="absolute opacity-0" :id="'tab-'+item.etag" name="tabs">
            <label class="block p-5 leading-normal cursor-pointer" :for="'tab-'+item.etag">{{ item.snippet.title }}</label>
            <div class="tab-content overflow-hidden border-l-2 bg-gray-100 border-blue-500 leading-normal">
              <div :id="'player-'+item.etag"></div>
            </div>
          </div>
        </div>
        <div class="mt-5">
          <form>
            <label for="per_page">Кол-во видео:</label>
            <input v-model="per_page" type="number" id="per_page" min="0" max="10">
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      query: '',
      items: [],
      per_page: 10,
      max_per_page: 10,
    }
  },

  mounted() {
    let tag = document.createElement('script');
    tag.src = "https://www.youtube.com/iframe_api";
    let firstScriptTag = document.getElementsByTagName('script')[0];
    firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
  },

  methods: {
    search() {
      this.per_page = this.per_page > this.max_per_page ? this.max_per_page: this.per_page <= 0 ? 1 : this.per_page;
      this.searched = this.query;
      this.$axios
        .$get('/search', {params: {
            q: this.query,
            count: this.per_page,
        }})
        .then(response => {
          console.log(response.items);
          this.items = response.items;
      })
    },

    openPlayer(item) {
      let player;
      player = new YT.Player('player-'+item.etag, {
        height: '360',
        width: '640',
        videoId: item.id.videoId,
        events: {
          'onReady': this.onPlayerReady,
          'onStateChange': this.onPlayerStateChange
        }
      });
      }
    },

    stopVideo() {
      player.stopVideo();
    },

    // Player events
    onPlayerReady(event) {
      event.target.playVideo();
    },

    onPlayerStateChange(event) {
      let done = false;
      if (event.data == YT.PlayerState.PLAYING && !done) {
        setTimeout(this.stopVideo, 6000);
        done = true;
      }
    },
}
</script>
<style>

.tab-content {
  max-height: 0;
  -webkit-transition: max-height .35s;
  -o-transition: max-height .35s;
  transition: max-height .35s;
}

.tab input:checked ~ .tab-content {
  max-height: 100vh;
}

.tab input:checked + label{
  font-size: 1.25rem;
  padding: 1.25rem;
  border-left-width: 2px;
  border-color: #0377fc;
  background-color: #f8fafc;
  color: #0377fc;
}

.tab label::after {
  float:right;
  right: 0;
  top: 0;
  display: block;
  width: 1.5em;
  height: 1.5em;
  line-height: 1.5;
  font-size: 1.25rem;
  text-align: center;
  -webkit-transition: all .35s;
  -o-transition: all .35s;
  transition: all .35s;
}

.tab input[type=checkbox] + label::after {
  content: "+";
  font-weight:bold;
  border-width: 1px;
  border-radius: 9999px;
  border-color: #b8c2cc;
}
.tab input[type=radio] + label::after {
  content: "\25BE";
  font-weight:bold;
  border-width: 1px;
  border-radius: 9999px;
  border-color: #b8c2cc;
}
/* Icon formatting - open */
.tab input[type=checkbox]:checked + label::after {
  transform: rotate(315deg);
  background-color: #0377fc;
  color: #f8fafc;
}
.tab input[type=radio]:checked + label::after {
  transform: rotateX(180deg);
  background-color: #0377fc;
  color: #f8fafc;
}

input[type='number']{
  width: 4  em;
}
</style>
