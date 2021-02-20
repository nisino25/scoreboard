<template>
  <div style="text-align:center" >
    <input type="date" v-model="beforeEdit" ><br>
  </div>
    
  <br>
  <div style="text-align:center">
    <button @click="getScores('yesterday')">Yesterday</button> &nbsp;
    <button @click="getScores('today')">Today</button>&nbsp;
    <button @click="getScores('tomorrow')">Tomorrow</button>

  </div>
  
  <br><br>


  <div v-if="gameData && !(isFetchingData) && !(hasFailed)">
    <div v-for="(game, i) in gameData.games" :key="i" style="text-align:center">
      <img :src="game.hTeam.linkName" alt="" class="teamLogo">
      <strong>{{game.hTeam.triCode}}</strong>&nbsp; &nbsp;
      <small>{{game.hTeam.score}}</small>&nbsp;
      &nbsp;<small :style="game.colorOfTheString">{{game.gameStatus}}</small>&nbsp;
      &nbsp;<small>{{game.vTeam.score}}</small>&nbsp;&nbsp; &nbsp; 
      <img :src="game.vTeam.linkName" alt="" class="teamLogo">
      <strong>{{game.vTeam.triCode}}</strong>&nbsp; 
      
      <hr>
      

    </div>
  </div>
  <div v-if="gameData &&gameData.games.length === 0 && !(isFetchingData)"><br><br><br> No Games</div>
  <div v-if="isFetchingData" style="text-align:center">
    <br><br><br>
    <strong>Loading.....</strong>
  </div>
  <div v-if="hasFailed && !(isFetchingData)" style="text-align:center"><br><br><br>
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

      updateCount:0,




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
      // console.log('Calling API')
      // console.log(URL)


      const res = await fetch(URL)
      const json = await res.json()
      console.log(json)
      if(json.Message !== 'Object not found.'){
        this.hasFailed = false
        this.gameData = json 
        this.isFetchingData = false

        let i =0
        let beforeCal = undefined;
        let pureTime = undefined
        let afterCal = undefined;
        let editedString = undefined;
        let getAMPMString = undefined;
        // console.log('------------')

        

        while(i < this.gameData.games.length){
          this.gameData.games[i].colorOfTheString = "color: black"
          if(this.gameData.games[i].period.current === 0){
            this.gameData.games[i].colorOfTheString === 'black'
            beforeCal = Number(this.gameData.games[i].startTimeEastern.split(':')[0]) - 3;
            pureTime =  Number(this.gameData.games[i].startTimeEastern.split(':')[0]) ;
            if(pureTime<=3 && pureTime >=0){
              afterCal = beforeCal + 12;
              getAMPMString = 'AM'
              if(beforeCal === 0){
                getAMPMString = 'PM'
                console.log('you called me ')
              }
            }else if(beforeCal === 9){
              afterCal = beforeCal
              console.log('jhue')
              getAMPMString = 'AM'
            }else{
              afterCal = beforeCal
              getAMPMString =this.gameData.games[i].startTimeEastern.substring(this.gameData.games[i].startTimeEastern.indexOf(" " )+ 1) 
              getAMPMString = getAMPMString.split(' ')[0];
            }
            editedString = this.gameData.games[i].startTimeEastern;
            editedString = editedString.substring(editedString.indexOf(":" )+ 1) 
            editedString = editedString.split(' ')[0];
            this.gameData.games[i].startTimePST  = String(afterCal) + ':' + editedString + ' '+ getAMPMString + ' PST'
            this.gameData.games[i].gameStatus = String(afterCal) + ':' + editedString + ' '+ getAMPMString + ' PST'

          }else if(this.gameData.games[i].period.isHalftime){
            this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = 'Half Time'
            
          }else if(this.gameData.games[i].period.current >=1 && this.gameData.games[i].period.current <=4 && this.gameData.games[i].isGameActivated && !this.gameData.games[i].period.isEndOfPeriod){
            this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = this.gameData.games[i].period.current + 'Q  ' + this.gameData.games[i].clock

          }else if(this.gameData.games[i].period.current >=1 && this.gameData.games[i].period.current <=3 && this.gameData.games[i].period.isEndOfPeriod){
             this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = 'End of ' + this.gameData.games[i].period.current + 'Q ' 

          }else if(this.gameData.games[i].isGameActivated && this.gameData.games[i].period.current >= 4){
             this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = String(this.gameData.games[i].period.current -4) + 'OT' + this.gameData.games[i].clock

          }else if(!this.gameData.games[i].isGameActivated && this.gameData.games[i].period.current === 4){
            this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = 'Final'

          }else if(!this.gameData.games[i].isGameActivated && this.gameData.games[i].period.current > 4){
            this.gameData.games[i].colorOfTheString = 'color: red'
            this.gameData.games[i].gameStatus = String(this.gameData.games[i].period.current -4) + ' OT Final'

          }else{
            // console.log('missed')
          }

          
          
          switch(this.gameData.games[i].hTeam.triCode){
            case 'NOP':
              this.gameData.games[i].hTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/NO.svg';
              break;

            case 'GSW':
              this.gameData.games[i].hTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/GS.svg';
              break;
            
            case 'PHX':
              this.gameData.games[i].hTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/PHO.svg';
              break;

            case 'NYK':
              this.gameData.games[i].hTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/NY.svg';
              break;

            case 'SAS':
              this.gameData.games[i].hTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/SA.svg';
              break;
            default:
              this.gameData.games[i].hTeam.linkName = `https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/` + this.gameData.games[i].hTeam.triCode + `.svg`
              break;
          }

          switch(this.gameData.games[i].vTeam.triCode){
            case 'NOP':
              this.gameData.games[i].vTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/NO.svg';
              break;

            case 'GSW':
              this.gameData.games[i].vTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/GS.svg';
              break;
            
            case 'PHX':
              this.gameData.games[i].vTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/PHO.svg';
              break;

            case 'NYK':
              this.gameData.games[i].vTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/NY.svg';
              break;

            case 'SAS':
              this.gameData.games[i].vTeam.linkName = 'https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/SA.svg';
              break;
            default:
              this.gameData.games[i].vTeam.linkName = `https://sportsfly.cbsistatic.com/fly-152/bundles/sportsmediacss/images/team-logos/nba/alt/` + this.gameData.games[i].vTeam.triCode + `.svg`
              break;
          }




           



          i++
        }

        // console.log(this.gameData.games)
      }else{
        this.hasFailed = true
        this.isFetchingData = false;
      }

    
     
      // console.log(this.gameData.games.length)
      
    },
    checkUpdate(){
      let checkDate = undefined

      let now = new Date();
      let y = now.getFullYear();
      let m = now.getMonth() + 1;
      let d = now.getDate();
      let mm = m < 10 ? '0' + m : m;
      let dd = d < 10 ? '0' + d : d;
      checkDate = '' + y + mm + dd;

      if(this.searchDate === checkDate){
        this.getScores()
        this.updateCount++;
        console.log(`Update ${this.updateCount}`)
      }else{
        console.log('wrong')
      }

    }
  },
  created() {
    this.interval = setInterval(() => this.checkUpdate(), 30000);  
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
  height: 23px;
  width: auto;
  margin-right: 6px;
  margin-top: 2px;
}
</style>
