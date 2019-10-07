<template>
<!-- eslint-disable -->
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
            Use Celsius <input type="checkbox" id="checkbox" v-model="isCelsius"><br/>
            <button type="button" class="success button" v-on:click="generateWeather()">Generate Weather</button>
        </div>

      </div>
      <div v-if="forecast.length > 0"> 
          <table>
            <thead>
              <tr>
                <th class="weatherTable">Day #</th>
                <th class="weatherTable">Time of Day</th>
                <th class="weatherTable">Cloudcover</th>
                <th class="weatherTable">Precipitation</th>
                <th class="weatherTable">Wind MPH</th>
                <th class="weatherTable">Degrees</th>
                <th class="weatherTable">Wind Chill</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="day in forecast" >
                <td class="weatherTable">{{day.number}}</td>
                <td class="weatherTable">{{day.time}}</td>
                <td class="cloudCover weatherTable">{{day.cloudCover}}</td>
                <td class="weatherTable ">{{day.precip}}</td>
                <td class="weatherTable">{{day.windSpeed}}</td>
                <td class="weatherTable">{{day.temp}}</td>
                <td class="weatherTable"></td> 
              </tr>
            </tbody>
          </table>
      </div>
    </div>
    <div> Generation rules from Ultimate Wilderness © 2017, Paizo Inc.. License: <a href="OGL.txt">OGL v1.0a</a> . Send suggestions and bug reports to: direrat at gmail.com</div>
  
    
  </div>
</template>

<style>
.cloudCover {
  text-align:left;
}
.weatherTable {
  border-right-style: solid;
}
</style>

<script>
/* eslint-disable */ 
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
      isCelsius: false,
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
      this.forecast = [];
      var chosenClimate;
      this.currentBaseLine = {};
      var precipDetails = this.getPrecipationInfo();
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
      switch(this.elevation) {
        case "seaLevel" :
          this.currentBaseLine.baseTemp = this.currentBaseLine.baseTemp + this.elevations[0].baseLineTempVar;
          break;
        case "lowland":
          this.currentBaseLine.baseTemp = this.currentBaseLine.baseTemp + this.elevations[1].baseLineTempVar;
          break;
        case "highland": 
          this.currentBaseLine.baseTemp = this.currentBaseLine.baseTemp + this.elevations[2].baseLineTempVar;
          break;
      }

      console.log("adjusted temp"+ this.currentBaseLine.baseTemp);
      var durationTotal = 0;
      var currentChance = 0;
      var variation;
      var holdDay = {};

      for(var i = 1; i <= this.days; i++) {
        var day = {};
        var night = {};
        if (durationTotal > 0) {
          
          day.number = i;
          day.time = "Day";
          day.temp = holdDay.temp;
          this.forecast.push(day);
          day.windSpeed = holdDay.windSpeed;
          day.cloudCover = holdDay.cloudCover;

          night.number = i;
          night.time = "Night";
          night.temp = day.temp - this.generateNightDrop();
          night.windSpeed = this.generateWindSpeed();
          night.cloudCover = day.cloudCover;
          this.forecast.push(night);
          durationTotal--;

        } else {

         /* Overcast conditions without precipitation increase the temperature in fall and winter by 10° F and decrease the temperature in spring and summer by the same amount. If precipitation occurs, the cloud cover functions as overcast.*/
          //2d6+3  night drop
          day.number = i;
          day.time = "Day";
          currentChance = this.generateRandom(100);
          day.windSpeed = this.generateWindSpeed();
          variation = this.getVariationFromClimate(chosenClimate, currentChance); 
          durationTotal = (Math.floor(Math.random() * (+variation.durationDice - +1)) + 1) + variation.durationAddition;
          holdDay = day;
          console.log("DurationTotal = "+ durationTotal)
          day.temp = this.currentBaseLine.baseTemp + this.getTempVariance(variation);
          day.cloudCover = this.generateCloudCover();

          this.forecast.push(day);

          night.number = i;
          night.time = "Night";
          night.temp = day.temp - this.generateNightDrop();
          night.windSpeed = this.generateWindSpeed();
          night.cloudCover = this.generateCloudCover();
          this.forecast.push(night);

        }
      } 
    },
    generatePrecip(temp, intensity, freq) {
      var rainValues;
      var isRain = false;
      var rainChance = this.generateRandom(100);
      switch(freq) {
        case "Drought":
          if(rainChance <= 5) {
            isRain = true;
          }
          break;
        case "Rare":
          if(rainChance <= 15) {
            isRain = true;
          }
          break;
        case "Intermittent":
          if(rainChance <= 30) {
            isRain = true;
          }
          break;
        case "Common":
          if(rainChance <= 60) {
            isRain = true;
          }
          break;
        case "Constant":
          if(rainChance <= 95) {
            isRain = true;
          }
          break;
      }
      
      if(isRain) {
        var precipType = this.generateRandom(100);
        if(temp < 32) {
          switch(intensity) {
            case "Light" :
              if(precipType>= 1 && precipType <=20){
                rainValues.type = "Light Fog"
                rainValues.duration = this.generateRandom(6) + "Hours";
              } else if(precipType>= 21 && precipType <=40) {
                rainValues.type = "Light Fog"
                rainValues.duration = this.generateRandom(8) + "Hours";
              } else if(precipType>= 41 && precipType <=50) {
                rainValues.type = "Medium Fog"
                rainValues.duration = this.generateRandom(4) + "Hours";
              } else if(precipType>= 51 && precipType <=60) {
                rainValues.type = "Light snow"
                rainValues.duration = "1 Hour";
              } else if(precipType>= 61 && precipType <=75) {
                rainValues.type = "Light Snow"
                rainValues.duration = this.generateRandom(4) + "Hours";
              } else if(precipType>= 76 && precipType <=100) {
                rainValues.type = "Light Snow"
                rainValues.duration = (this.generateRandom(12) + this.generateRandom(12))+ "Hours";
              }
              break;
            case "Medium":
              if(precipType>= 1 && precipType <=10){
                rainValues.type = "Medium Fog"
                rainValues.duration = this.generateRandom(6) + "Hours";
              } else if(precipType>= 11 && precipType <=20) {
                rainValues.type = "Medium Fog"
                rainValues.duration = this.generateRandom(8) + "Hours";
              } else if(precipType>= 21 && precipType <=30) {
                rainValues.type = "Heavy Fog"
                rainValues.duration = this.generateRandom(4) + "Hours";
              } else if(precipType>= 31 && precipType <=50) {
                rainValues.type = "Medium snow"
                rainValues.duration = this.generateRandom(4) + "Hours";
              } else if(precipType>= 51 && precipType <=90) {
                rainValues.type = "Medium Snow"
                rainValues.duration = this.generateRandom(8) + "Hours";
              } else if(precipType>= 91 && precipType <=100) {
                rainValues.type = "Medium Snow"
                rainValues.duration = (this.generateRandom(12) + this.generateRandom(12))+ "Hours";
              }
              break;
            case "Heavy":
              if(precipType>= 1 && precipType <=10){
                rainValues.type = "Medium Fog"
                rainValues.duration = this.generateRandom(8) + " Hours";
              } else if(precipType>= 11 && precipType <=20) {
                rainValues.type = "Heavy Fog"
                rainValues.duration = (this.generateRandom(6)+this.generateRandom(6)) + " Hours";
              } else if(precipType>= 21 && precipType <=60) {
                rainValues.type = "Light Snow"
                rainValues.duration = (this.generateRandom(12) + this.generateRandom(12))+ " Hours";
              } else if(precipType>= 61 && precipType <=90) {
                rainValues.type = "Medium snow"
                rainValues.duration = this.generateRandom(8) + " Hours";
              } else if(precipType>= 91 && precipType <=100) {
                rainValues.type = "Heavy Snow"
                rainValues.duration = this.generateRandom(6) + " Hours";
              } 
              break;
            case "Torrential":
              if(precipType>= 1 && precipType <=5){
                rainValues.type = "Heavy Fog"
                rainValues.duration = this.generateRandom(8) + " Hours";
              } else if(precipType>= 6 && precipType <=10) {
                rainValues.type = "Heavy Fog"
                rainValues.duration = (this.generateRandom(6)+this.generateRandom(6)) + " Hours";
              } else if(precipType>= 11 && precipType <=50) {
                rainValues.type = "Heavy Snow"
                rainValues.duration = this.generateRandom(4) + " Hours";
              } else if(precipType>= 51 && precipType <=90) {
                rainValues.type = "Heavy snow"
                rainValues.duration = this.generateRandom(8) + " Hours";
              } else if(precipType>= 91 && precipType <=100) {
                rainValues.type = "Heavy Snow"
                rainValues.duration = (this.generateRandom(12) + this.generateRandom(12))+ " Hours";
              } 
              break;
          }
        } else {
          switch(intensity) {
            case "Light" :
              break;
            case "Medium":
              break;
            case "Heavy":
              break;
            case "Torrential":
              break;
          }
        }
      }
    },
    generateCloudCover() {
      var chance = this.generateRandom(100); 
      var cloudCover = "";
      if(chance >=1 && chance <= 50) {
          cloudCover = "None";
      } else if(chance >=51 && chance <= 70 ) {
          cloudCover = "Light Clouds";
      } else if(chance >=71 && chance <= 85) {
          cloudCover = "Mediucm Clouds";
      } else if(chance >=86 && chance <= 100) {
        cloudCover = "WindStorm";
      } 
      return cloudCover;

    },
    generateWindSpeed() {
      var chance = this.generateRandom(100); 
      var windSpeed = 0;
      if(chance >=1 && chance <= 50) {
          windSpeed = Math.floor(Math.random() * (+10 - +0)) + 0;
      } else if(chance >=51 && chance <= 80 ) {
          windSpeed = Math.floor(Math.random() * (+20 - +11)) + 11;
      } else if(chance >=81 && chance <= 90) {
        windSpeed = Math.floor(Math.random() * (+30 - +21)) + 21;
      } else if(chance >=91 && chance <= 95) {
        windSpeed = Math.floor(Math.random() * (+50 - +31)) + 31;
      } else if(chance >=96 && chance <= 100) {
        windSpeed = Math.floor(Math.random() * (+80 - +51)) + 51;
      }
      return windSpeed;

    },
    generateNightDrop() {
      //2d6+3
      var nightDrop = 0;
      for(var i = 0; i< 2; i++) {
        nightDrop += this.generateRandom(6);
      }
      nightDrop += 3;
      return nightDrop;
    },
    getTempVariance(variation) {
      var tempChange = 0;
      if(variation.numDice > 0 ) {
          for(var i = 0; i <variation.numDice; i++) {
            if(variation.variation == "negative") {
              tempChange -= this.generateRandom(variation.diceType);
            } else {
              tempChange += this.generateRandom(variation.diceType);
            }
            
          }
      }
      return tempChange;
      

    },
    getVariationFromClimate(chosenClimate, currentChance) {
      var variation;
      for(var i = 0; i< chosenClimate.variations.length; i++) {
          variation = chosenClimate.variations[i];
          if(currentChance >= variation.chanceLow && currentChance <= variation.chanceHigh) {
            return variation;
          }
      }
    },
    getPrecipationInfo() {
      /*Find frequency baseline
    depends on season
    Climate will adjust
    elevation will adjust this down
find intensity baseline
    set by elevation
    adjusted by climate*/
      //Drought	,Rare	,Intermittent, Common, Constant
      var precipDetails = {};
      switch(this.season) {
        case "winter" :
          if(this.climate == "tropical") {
            precipDetails.freq = "Intermittent";
          } else if(this.climate == "cold") {
            precipDetails.freq = "Drought";
          } else {
            precipDetails.freq = "Rare";
          }
          break;
        case "spring" :
          if(this.climate == "tropical") {
            precipDetails.freq = "Constant";
          } else if(this.climate == "cold") {
            precipDetails.freq = "Rare";
          }else {
            precipDetails.freq = "Intermittent";
          }
          break;
        case "summer" :
          if(this.climate == "tropical") {
            precipDetails.freq = "Common";
          } else if(this.climate == "cold") {
            precipDetails.freq = "Intermittent";
          }else {
            precipDetails.freq = "Common";
          }
          break;
        case "fall" :
          if(this.climate == "tropical") {
            precipDetails.freq = "Constant";
          } else if(this.climate == "cold") {
            precipDetails.freq = "Rare";
          }else {
            precipDetails.freq = "Intermittent";
          }
          break;
        default:
          console.log("No season selected");
      }
      //Drought	,Rare	,Intermittent, Common, Constant
      if(this.elevation == "highland") {
        switch(precipDetails.freq) {
          case "Constant":
            precipDetails.freq = "Common";
            break;
          case "Commont":
            precipDetails.freq = "Intermittent";
            break;
          case "Intermittent":
            precipDetails.freq = "Rare";
            break;
          case "Rare":
            precipDetails.freq = "Drought";
            break;
        }
      }
      //light, medium, heavy, torrential
      switch(this.elevation) {
        case "seaLevel" :
          if(this.climate == "cold") {
            precipDetails.intensity = "Medium"
          } else if (this.climate == "tropical") {
            precipDetails.intensity = "Torrential"
          } else {
            precipDetails.intensity = "Heavy"
          }
          break;
        case "lowland":
          if(this.climate == "cold") {
            precipDetails.intensity = "Light"
          } else if (this.climate == "tropical") {
            precipDetails.intensity = "Heavy"
          } else {
            precipDetails.intensity = "Medium"
          }
          break;
        case "highland": 
          if(this.climate == "cold") {
            precipDetails.intensity = "Light"
          } else if (this.climate == "tropical") {
            precipDetails.intensity = "Torrential"
          } else {
            precipDetails.intensity = "Medium"
          }
      }
      switch(precipDetails.freq) {
        case "Constant":
          precipDetails.freqChance = "95";
          break;
        case "Commont":
          precipDetails.freqChance = "60";
          break;
        case "Intermittent":
          precipDetails.freqChance = "30";
          break;
        case "Rare":
          precipDetails.freqChance = "15";
          break;
        case "Drought":
          precipDetails.freqChance = "5";
          break;
          
      }


    },
    generateRandom: function(ceiling) {
      var newRandom = Math.floor(Math.random() * ceiling);
      newRandom += 1;
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
        this.elevations.push({name: "Highland", altitudeRange: 'Below 1,000 ft.', baseLineTempVar: -10, baseLinePrecipIntensity: 'medium', baseLinePrecipFreq: 'decrease'});
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
