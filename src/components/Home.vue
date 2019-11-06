<template>
  <div>
    <h1>Pokemon Go 101</h1>

    <div class="container">
      <div class="m-2 p-2">
        <b-form-input v-model="pokemonSearch" v-on:keyup="doDebounce" placeholder="Search Pokemon"></b-form-input>
      </div>

      <div v-if="pokemon.length === 0" class="text-center" style="margin-top: 25%">
        <b-spinner type="grow" label="Spinning"></b-spinner>
      </div>

      <div v-if="!pokemonSearch" class="row">
        <div
          class="pokemonContainer border p-2 col-lg-2 col-md-3 col-sm-4 col-6"
          v-for="animal in pokemon"
          v-bind:key="animal.id"
          @click="goToDetailsPage(animal)"
        >
          <h4>{{animal.name}}</h4>
          <img :src="getImgUrl(animal)" alt="animal.name" height="150" width="150" />
          <h6 style="position:absolute; right:10px; bottom: 0px">{{("000" + animal.id).slice(-3)}}</h6>
        </div>
      </div>

      <div v-else class="row">
        <div v-if="pokemonSearchResults.length === 0">No results found.</div>
        <div
          class="pokemonContainer border p-2 col-lg-2 col-md-3 col-sm-4 col-6"
          v-for="animal in pokemonSearchResults"
          v-bind:key="animal.id"
          @click="goToDetailsPage(animal)"
        >
          <h4>{{animal.name}}</h4>
          <img :src="getImgUrl(animal)" alt="animal.name" height="150" width="150" />
          <h6 style="position:absolute; right:10px; bottom: 0px">{{("000" + animal.id).slice(-3)}}</h6>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
var debounce = require("debounce");

export default {
  name: "Home",
  props: {
    msg: String
  },
  data: function() {
    return {
      pokemon: [],
      pokemonObj: {},
      pokemonSearch: "",
      pokemonSearchResults: []
    };
  },
  mounted() {
    this.getAllPokemon();
  },
  methods: {
    getAllPokemon: function() {
      fetch("https://pokemon-go1.p.rapidapi.com/released_pokemon.json", {
        method: "GET",
        headers: {
          "x-rapidapi-host": "pokemon-go1.p.rapidapi.com",
          "x-rapidapi-key": "c5e9119c84msh6c5eb83ff319aa0p164365jsn340d3ca81561"
        }
      })
        .then(resp => resp.json())
        .then(data => {
          const listOfObj = Object.keys(data).map(key => {
            return data[key];
          });
          this.pokemon = listOfObj;

          if (localStorage.allPokemon) {
            this.pokemonObj = JSON.parse(localStorage.getItem("allPokemon"));
          } else {
            this.pokemonObj = data;
            this.getRestOfPokemonData();
          }
        });
    },
    getRestOfPokemonData() {
      const urls = [
        "https://pokemon-go1.p.rapidapi.com/pokemon_max_cp.json",
        "https://pokemon-go1.p.rapidapi.com/shiny_pokemon.json",
        "https://pokemon-go1.p.rapidapi.com/alolan_pokemon.json",
        "https://pokemon-go1.p.rapidapi.com/pokemon_types.json",
        "https://pokemon-go1.p.rapidapi.com/pokemon_candy_to_evolve.json",
        "https://pokemon-go1.p.rapidapi.com/pokemon_buddy_distances.json",
        "https://pokemon-go1.p.rapidapi.com/pokemon_stats.json",
        "https://pokemon-go1.p.rapidapi.com/pokemon_encounter_data.json"
      ];
      Promise.all(
        urls.map((url, index) => {
          fetch(url, {
            headers: {
              "x-rapidapi-host": "pokemon-go1.p.rapidapi.com",
              "x-rapidapi-key":
                "c5e9119c84msh6c5eb83ff319aa0p164365jsn340d3ca81561"
            }
          })
            .then(resp => resp.json())
            .then(data => {
              switch (index) {
                case 0:
                  this.addAnArray(data, "cps");
                  break;
                case 1:
                  this.addShiny(data);
                  break;
                case 2:
                  this.addAlolan(data);
                  break;
                case 3:
                  this.addAnArray(data, "types");
                  break;
                case 4:
                  this.addCandy(data);
                  break;
                case 5:
                  this.addBuddyDistance(data);
                  break;
                case 6:
                  this.addAnArray(data, "stats");
                  break;
                case 7:
                  this.addAnArray(data, "encounters");
                  break;
              }
              localStorage.setItem(
                "allPokemon",
                JSON.stringify(this.pokemonObj)
              );
            });
        })
      );
    },
    addShiny(data) {
      for (const shiny in data) {
        this.pokemonObj[shiny]["shiny"] = data[shiny];
      }
    },
    addAlolan(data) {
      for (const alolan in data) {
        this.pokemonObj[alolan]["alolan"] = data[alolan];
      }
    },
    addCandy(data) {
      for (const amount in data) {
        data[amount].forEach(candy => {
          if (this.pokemonObj[candy.pokemon_id]["candy"]) {
            this.pokemonObj[candy.pokemon_id]["candy"].push(candy);
          } else {
            this.pokemonObj[candy.pokemon_id]["candy"] = [candy];
          }
        });
      }
    },
    addBuddyDistance(data) {
      for (const distance in data) {
        data[distance].forEach(distance => {
          const pokemon = this.pokemonObj[distance.pokemon_id];
          if (pokemon) {
            if (this.pokemonObj[distance.pokemon_id]["buddyDistance"]) {
              this.pokemonObj[distance.pokemon_id]["buddyDistance"].push(
                distance
              );
            } else {
              this.pokemonObj[distance.pokemon_id]["buddyDistance"] = [
                distance
              ];
            }
          }
        });
      }
    },
    addAnArray(data, label) {
      data.forEach(item => {
        const pokemon = this.pokemonObj[item.pokemon_id];
        if (pokemon) {
          if (pokemon[label]) {
            pokemon[label].push(item);
          } else {
            pokemon[label] = [item];
          }
        }
      });
    },
    getImgUrl(animal) {
      let id = ("000" + animal.id).slice(-3);
      if ([412].indexOf(animal.id) !== -1)
        return require("./../assets/pokemon_icons/pokemon_icon_" +
          id +
          "_11.png");
      return require("./../assets/pokemon_icons/pokemon_icon_" +
        id +
        "_00.png");
    },
    goToDetailsPage(animal) {
      this.$router.push({
        path: `details/${animal.id}`
      });
    },
    filterPokemon() {
      debounce(function() {
        console.log("bounce");
      }, 0);
    },
    doDebounce: debounce(function() {
      let newSearch = this.pokemon.filter(
        pokemon =>
          pokemon.name
            .toLowerCase()
            .indexOf(this.pokemonSearch.toLowerCase()) !== -1
      );
      this.pokemonSearchResults = newSearch;
    }, 250)
  }
};
</script>

<style scoped>
h1 {
  margin: 40px 0 20px 0;
  text-align: center;
}
a {
  color: #42b983;
}
.pokemonContainer {
  background-color: rgba(255, 255, 255, 0.6);
}
.pokemonContainer:hover {
  transform: perspective(500px) scaleZ(1) rotateX(5deg);
  background-color: rgba(255, 255, 255, 0.5);
  transition: 0.5s ease;
  cursor: pointer;
}
</style>
