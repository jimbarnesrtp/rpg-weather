<template>
  <div id="app">
    <img src="images/header.jpg" /><br/>
    With this app you can generate, save and print fantasy weather for up to 30 days.
    <br/>
    <div class="grid-container fluid">
      <div class="grid-x align-center">
        <div class="cell medium-2">
            <select v-model="climate">
              <option value="">Choose Climate</option>
              <option value="cold">Cold</option>
              <option value="temperate">Temperate</option>
              <option value="tropical">Tropical</option>
            </select>
        </div>
        <div class="cell small-2">
            <select v-model="elevation">
              <option value="">Choose Elevation</option>
              <option value="seaLevel">Sea Level</option>
              <option value="lowland">Lowland</option>
              <option value="highland">Highland</option>
            </select>
        </div>
        <div class="cell small-2">
            <select v-model="season">
              <option value="">Choose Season</option>
              <option value="winter">Winter</option>
              <option value="spring">Spring</option>
              <option value="summer">Summer</option>
              <option value="fall">Fall</option>
            </select>
        </div>
        <div class="cell small-2" v-if="climate == 'cold'">
            <select v-model="pole">
              <option value="">Distance Pole</option>
              <option value="outside">&gt; 500</option>
              <option value="close">500 - 250</option>
              <option value="veryClose"> &lt; 250</option>
            </select>
        </div>
        <div class="cell small-1" >
            <select v-model="days">
              <option value="">Days</option>
              <option value="1">1</option>
              <option value="2">2</option>
              <option value="3">3</option>  
              <option value="4">4</option>    
              <option value="7">7</option> 
              <option value="10">10</option>   
              <option value="14">14</option>  
              <option value="20">20</option>  
              <option value="21">21</option>
              <option value="28">28</option>
              <option value="30">30</option>
            </select>
        </div>
      </div>
      <div class="grid-x align-center">
         <div class="cell medium-3">
            <button type="button" class="success button" v-on:click="generateWeather()">Generate Weather</button>
        </div>

      </div>
      <div v-if="forecast.length > 0"> 
          <table>
            <th>Day #</th><th>Time of Day</th><th>Cloudcover</th><th>Wind MPH</th><th>Degrees</th><th>Wind Chill</th>
             <tr v-for="day in forecast" ><td>{{day.number}}</td><td>{{day.time}}</td><td></td><td></td><td>{{day.temp}}</td><td></td> </tr>
          </table>
      </div>
    </div>
    <div> Generation rules from Ultimate Wilderness © 2017, Paizo Inc.. License: <a href="OGL.txt">OGL v1.0a</a> . Send suggestions and bug reports to: direrat at gmail.com</div>
  
    
  </div>
</template>

<script>
export default {
  data() {
    return {
      climate: "",
      elevation: "",
      season: "",
      pole: "",
      days: "",
      forecast: [],
      climates: [],
      elevations: [],
      currentBaseLine: {}

    };
  },
  created: function() {
    this.loadData();
  },
  methods: {
    calculateCelsius: function(temp) {
      var newTemp = (temp - 32) / 1.8;
      return newTemp;
    },
    generateWeather: function() {
      var chosenClimate;
      this.currentBaseLine = {};
      if(this.climate == "cold") {
        chosenClimate = this.climates[0];
      } else if(this.climate == "temperate") {
        chosenClimate = this.climates[1];
      } else if(this.climate == "tropical") {
        chosenClimate = this.climates[2];
      }
      console.log("here");
      switch(this.season) {
        case "winter" :
          this.currentBaseLine.baseTemp = chosenClimate.winterTemp;
          console.log("baseTEmp:"+ this.currentBaseLine.baseTemp)
          if(this.pole == "close") {
            this.currentBaseLine.baseTemp = this.currentBaseLine.baseTemp - 10;
          } else if(this.pole == "veryClose") {
            this.currentBaseLine.baseTemp = this.currentBaseLine.baseTemp - 20;
          }
          break;
        case "spring" :
          this.currentBaseLine.baseTemp = chosenClimate.springTemp;
          break;
        case "summer" :
          this.currentBaseLine.baseTemp = chosenClimate.summerTemp;
          break;
        case "fall" :
          this.currentBaseLine.baseTemp = chosenClimate.fallTemp;
          break;
        default:
          console.log("No season selected");
      }

        
    },
    generateRandom: function(ceiling) {
      var newRandom = Math.floor(Math.random() * ceiling);
      newRandom +1;
      return newRandom;
    },
    loadData: function() {
        this.climates=[];
        var cold = {name: "Cold", winterTemp: 20, springTemp: 30, summerTemp: 40, fallTemp: 30, precipAdj: 'down', variations: []};
        cold.variations.push({chanceLow: 1, chanceHigh: 20, variation: 'negative', numDice: 3, diceType: 10, durationDice: 4, durationAddition: 0});
        cold.variations.push({chanceLow: 21, chanceHigh: 40, variation: 'negative', numDice: 2, diceType: 10, durationDice: 6, durationAddition: 1});
        cold.variations.push({chanceLow: 41, chanceHigh: 60, variation: 'negative', numDice: 1, diceType: 10, durationDice: 6, durationAddition: 2});
        cold.variations.push({chanceLow: 61, chanceHigh: 80, variation: 'none', numDice: 0, diceType: 0, durationDice: 4, durationAddition: 2});
        cold.variations.push({chanceLow: 81, chanceHigh: 95, variation: 'positive', numDice: 1, diceType: 10, durationDice: 6, durationAddition: 1});
        cold.variations.push({chanceLow: 96, chanceHigh: 99, variation: 'positive', numDice: 2, diceType: 10, durationDice: 4, durationAddition: 0});
        cold.variations.push({chanceLow: 100, chanceHigh: 100, variation: 'positive', numDice: 3, diceType: 10, durationDice: 2, durationAddition: 0});
        this.climates.push(cold);

        var temperate = {name: "Temperate", winterTemp: 30, springTemp: 60, summerTemp: 80, fallTemp: 60, precipAdj: '', variations: []};
        temperate.variations.push({chanceLow: 1, chanceHigh: 5, variation: 'negative', numDice: 3, diceType: 10, durationDice: 2, durationAddition: 0});
        temperate.variations.push({chanceLow: 6, chanceHigh: 15, variation: 'negative', numDice: 2, diceType: 10, durationDice: 4, durationAddition: 0});
        temperate.variations.push({chanceLow: 16, chanceHigh: 35, variation: 'negative', numDice: 1, diceType: 10, durationDice: 4, durationAddition: 1});
        temperate.variations.push({chanceLow: 36, chanceHigh: 65, variation: 'none', numDice: 0, diceType: 0, durationDice: 6, durationAddition: 1});
        temperate.variations.push({chanceLow: 66, chanceHigh: 85, variation: 'positive', numDice: 1, diceType: 10, durationDice: 4, durationAddition: 1});
        temperate.variations.push({chanceLow: 88, chanceHigh: 95, variation: 'positive', numDice: 2, diceType: 10, durationDice: 4, durationAddition: 0});
        temperate.variations.push({chanceLow: 96, chanceHigh: 100, variation: 'positive', numDice: 3, diceType: 10, durationDice: 2, durationAddition: 0});
        this.climates.push(temperate);

        var tropical = {name: "Tropical", winterTemp: 50, springTemp: 75, summerTemp: 95, fallTemp: 75, precipAdj: 'up', variations: []};
        tropical.variations.push({chanceLow: 1, chanceHigh: 10, variation: 'negative', numDice: 2, diceType: 10, durationDice: 2, durationAddition: 0});
        tropical.variations.push({chanceLow: 11, chanceHigh: 25, variation: 'negative', numDice: 1, diceType: 10, durationDice: 2, durationAddition: 0});
        tropical.variations.push({chanceLow: 26, chanceHigh: 55, variation: 'none', numDice: 0, diceType: 0, durationDice: 4, durationAddition: 0});
        tropical.variations.push({chanceLow: 56, chanceHigh: 85, variation: 'positive', numDice: 1, diceType: 10, durationDice: 4, durationAddition: 0});
        tropical.variations.push({chanceLow: 86, chanceHigh: 100, variation: 'positive', numDice: 2, diceType: 10, durationDice: 2, durationAddition: 0});
        this.climates.push(tropical);

        this.elevations = [];
        this.elevations.push({name: "Sea Level", altitudeRange: 'Below 1,000 ft.', baseLineTempVar: +10, baseLinePrecipIntensity: 'heavy', baseLinePrecipFreq: ''});
        this.elevations.push({name: "Lowland", altitudeRange: '1,000 ft. to 5,000 ft.', baseLineTempVar: 0, baseLinePrecipIntensity: 'medium', baseLinePrecipFreq: ''});
        this.elevations.push({name: "Highland", altitudeRange: 'Below 1,000 ft.', baseLineTempVar: +10, baseLinePrecipIntensity: 'medium', baseLinePrecipFreq: 'decrease'});
    }
    
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
