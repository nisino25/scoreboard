<template>
  <input type="text" v-model="searchDate" >
  <button @click="getScores()">Get the score of {{searchDate}} on console</button>


  <div v-if="gameData">
    <div v-for="(game, i) in gameData.games" :key="i">
      <strong>{{game.hTeam.triCode}}</strong>&nbsp; <small>{{game.hTeam.score}}</small>&nbsp;
      <!-- <strong>{{game.period.current}}</strong>&nbsp; -->
      <strong>{{game.vTeam.triCode}}</strong>&nbsp; <small>{{game.vTeam.score}}</small>&nbsp;&nbsp;
      <span style="color:red">{{game.period.current}}</span>
      <hr>

    </div>
  </div>
  
</template>

<script>

export default {
  name: 'App',
  data(){
    return{
      searchDate: undefined,
      isFetchingData: false,
      gameData: undefined,
      




    }
  },
  methods:{
    async getScores() {
      // this.showingMovie = true
      // this.decades = false;
      // this.pageForMyLists = false;
      // this.multipleYears = true;
      // this.multipleContents = false;
      // this.count =0
      // this.isImageLoaded = false
      // this.condition = 'searching'
      this.isFetchingData = true
      // const query = this.query


      const URL = 'http://data.nba.net/10s/prod/v1/'+ this.searchDate + '/scoreboard.json'
      // const URL = 'http://data.nba.net/10s/prod/v1/20210215/scoreboard.json'
      console.log('Calling API')
      console.log(URL)
      const res = await fetch(URL)
      const json = await res.json()
      console.log(json)
      this.gameData = json 
      console.log(this.gameData.games[0].hTeam.triCode)
      console.log(this.gameData.games[0].hTeam.score)
      
      this.isFetchingData = false
      // if(json.success === false){
      //   this.condition = 'error'
      // }else{
      //   this.condition = 'randomSuccess'
      // }
      // if (query !== this.query) {
      //   await this.searchMovie()
      // }
      
    },
  },
  mounted(){
    // console.log(Date())
    var now = new Date();
    var y = now.getFullYear();
    var m = now.getMonth() + 1;
    var d = now.getDate();
    var mm = m < 10 ? '0' + m : m;
    var dd = d < 10 ? '0' + d : d;
    this.searchDate  = '' + y + mm + dd


    this.getScores()
    // this.searchDate = '20210215'
  }

}
</script>

<style>
</style>
