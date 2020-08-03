<template>
  <div id="app">
    <button @click="toggleDebug">toggle debug</button> {{ debug }}<br/>
    <button @click="logpk">log</button><br/>
    <button @click="calculate">calculate</button><br/>
    <button @click="saveFile">save</button><br/>

    <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
  </div>
</template>

<script>
/* import HelloWorld from "./components/HelloWorld.vue"; */
import rawpokemon from "@/assets/pokedex.json";

export default {
  name: "App",
  components: {
    /* HelloWorld */
  },
  data() {
    return {
      debug: false,
      pokemon: {},
      fusions: []
    }
  },
  methods: {
    toggleDebug() {
      this.debug = !this.debug;
    },
    log(val) {
      if(this.debug) console.log(val);
    },
    logpk() {
      this.log(rawpokemon[0]);
      rawpokemon
        .filter(p => { return p.id < 649 }) // zum testen auf 20 beschränken, gibt wohl 649(?)
        .map(p => { this.pokemon[p.id] = p });

      this.log(this.pokemon[150]);
    },
    // gibt momentan nur pokemon mit overall > 470 zurück
    // TODO: parameterize
    //    calculate('bigger', 470)
    //    calculate('smaller', 100)
    calculate() {
      this.log(this.pokemon[99]);

      /*Object.keys(this.pokemon).map(function(key, index) {
        this.log(key, index);
        this.log(this.pokemon[key]);
      });*/

      Object.keys(this.pokemon).forEach(function (headId) { // headId, index
        // this.log(headId, index);
        Object.keys(this.pokemon).forEach(function (bodyId) {
          if(headId == bodyId) return;
          const variantOne = this.combine(headId, bodyId);
          const variantTwo = this.combine(bodyId, headId);
          
          if(variantOne !== undefined) this.fusions.push(variantOne);
          if(variantTwo !== undefined) this.fusions.push(variantTwo);
        }, this);
      }, this);
    },
    combine(headId, bodyId) {
      let fusion = {
        base: {},
        name: {},
        type: [],
        overallStats: 0
      };
      const headPokemon = this.pokemon[headId];
      const bodyPokemon = this.pokemon[bodyId];

      const headMultipliers = {
        'Attack': 1/3,
        'Defense': 2/3,
        'HP': 2/3,
        'Sp. Attack': 2/3,
        'Sp. Defense': 2/3,
        'Speed': 1/3
      };
      const bodyMultipliers = {
        'Attack': 2/3,
        'Defense': 2/3,
        'HP': 1/3,
        'Sp. Attack': 1/3,
        'Sp. Defense': 1/3,
        'Speed': 2/3
      };
      
      Object.keys(headPokemon.base).forEach(function (key){
        const keyValue = Math.round(
          (headPokemon.base[key] * headMultipliers[key]) +
          (bodyPokemon.base[key] * bodyMultipliers[key])
        );
        fusion.base[key] = keyValue;
        fusion.overallStats += keyValue;
      });

      fusion.headId = headId;
      fusion.bodyId = bodyId;

      fusion.name.english =
        this.pokemon[headId].name.english + ' + ' + this.pokemon[bodyId].name.english;
      
      fusion.type[0] = this.pokemon[headId].type[0];
      fusion.type[1] = this.pokemon[bodyId].type[1] || this.pokemon[bodyId].type[1];
      if(fusion.type[0] == fusion.type[1]) delete fusion.type[1];

      this.log(fusion.overallStats);
      if(fusion.overallStats > 680){
        return fusion
      } else {
        return undefined;
      }

      // return fusion;
    },
    saveFile() {
      const data = JSON.stringify(this.fusions);
      const blob = new Blob([data], {type: 'text/plain'});
      const e = document.createEvent('MouseEvents'),
            a = document.createElement('a');
      a.download = 'test.json';
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
