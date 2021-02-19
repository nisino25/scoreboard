<template>
  <input type="date" v-model="beforeEdit" ><br>
  <br>
  <button @click="getScores('yesterday')">Yesterday</button>
  <button @click="getScores('today')">Today</button>
  <button @click="getScores('tomorrow')">Tomorrow</button>
  <br><br>


  <div v-if="gameData && !(isFetchingData) && !(hasFailed)">
    <div v-for="(game, i) in gameData.games" :key="i">
      <img :src="`https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/` + game.hTeam.triCode + `.svg`" alt="" class="teamLogo">
      <strong>{{game.hTeam.triCode}}</strong>&nbsp; <small>{{game.hTeam.score}}</small>&nbsp;
      <!-- <strong>{{game.period.current}}</strong>&nbsp; -->
      <img :src="`https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/` + game.vTeam.triCode + `.svg`" alt="" class="teamLogo">
      <strong>{{game.vTeam.triCode}}</strong>&nbsp; <small>{{game.vTeam.score}}</small>&nbsp;&nbsp;
      <span style="color:red" v-if="!game.isGameActivated && game.period.current >= 4">FINAL</span>
      <span style="color:red" v-else-if="game.period.current !==0">{{game.period.current}}Q {{game.clock}}</span>
      <small  v-else-if="game.period.current === 0" >{{game.startTimeEastern}}</small>
      <hr>
      

    </div>
  </div>
  <div v-if="gameData &&gameData.games.length === 0 && !(isFetchingData)"><br><br><br> No Games</div>
  <div v-if="isFetchingData">
    <br><br><br>
    <strong>Loading.....</strong>
  </div>
  <div v-if="hasFailed && !(isFetchingData)"><br><br><br>
    Sorry the latest data you can get is 20141118
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
      displayDate: undefined,
      beforeEdit: undefined,



      hasFailed: false,





    }
  },
  methods:{
    async getScores(selectedDate) {

      switch(selectedDate){
        case 'yesterday':
          // var d = new Date();
          // d.setDate(d.getDate() - 1);

          var today = new Date()
          var yesterday = new Date(today)
          yesterday.setDate(yesterday.getDate() - 1)
          var y = yesterday.getFullYear();
          var m = yesterday.getMonth() + 1;
          var d = yesterday.getDate();
          var mm = m < 10 ? '0' + m : m;
          var dd = d < 10 ? '0' + d : d;
          this.searchDate  = '' + y + mm + dd

          // var now = new Date()
          // var y = now.getFullYear();
          // var m = now.getMonth() + 1;
          // var d = now.getDate();
          // var mm = m < 10 ? '0' + m : m;
          // var dd = d < 10 ? '0' + d : d;
          // this.searchDate  = '' + y + mm + dd

          break;

        case 'today':
          var now = new Date();
          y = now.getFullYear();
          m = now.getMonth() + 1;
          d = now.getDate();
          mm = m < 10 ? '0' + m : m;
          dd = d < 10 ? '0' + d : d;
          this.searchDate  = '' + y + mm + dd
          break;

        case 'tomorrow':
          today = new Date()
          var tomorrow = new Date(today)
          tomorrow.setDate(tomorrow.getDate() + 1)
          y = tomorrow.getFullYear();
          m = tomorrow.getMonth() + 1;
          d = tomorrow.getDate();
          mm = m < 10 ? '0' + m : m;
          dd = d < 10 ? '0' + d : d;
          this.searchDate  = '' + y + mm + dd
          break;

        case 'special':
          this.searchDate = this.beforeEdit.replace('-', '');
          this.searchDate = this.searchDate.replace('-', '');
        
        
      }
      this.isFetchingData = true


      const URL = 'https://data.nba.net/10s/prod/v1/'+ this.searchDate + '/scoreboard.json'
      // const URL = 'http://data.nba.net/10s/prod/v1/20210215/scoreboard.json'
      console.log('Calling API')
      console.log(URL)


      const res = await fetch(URL)
      const json = await res.json()
      console.log(json)
      if(json.Message !== 'Object not found.'){
        this.hasFailed = false
        this.gameData = json 
        this.isFetchingData = false
      }else{
        this.hasFailed = true
        this.isFetchingData = false;
      }
     
      // console.log(this.gameData.games.length)
      
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
  },
  watch:{
    beforeEdit: function(){
      this.getScores('special')
      console.log(this.beforeEdit)
    }
  },

}
</script>

<style>
.teamLogo{
  height: 20px;
  width: auto;
  margin-right: 5px;
}
</style>
