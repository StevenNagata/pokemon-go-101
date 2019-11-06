<template>
  <div class="container">
    <div @click="goHome()" class="backHome">
      <i class="center material-icons">arrow_back</i>
    </div>
    <div v-if="currentPokemon">
      <div id="carouselExampleControls" class="carousel slide" data-ride="carousel">
        <a
          v-if="parseInt(currentPokemon.id) > 1"
          @click="updatePokemon(parseInt(currentPokemon.id), 'previous')"
          class="carousel-control-prev"
          role="button"
          data-slide="prev"
        >
          <span class="carousel-control-prev-icon" aria-hidden="true"></span>
          <span class="sr-only">Previous</span>
        </a>

        <a
          v-if="parseInt(currentPokemon.id) < 809"
          @click="updatePokemon(parseInt(currentPokemon.id), 'next')"
          class="carousel-control-next"
          role="button"
          data-slide="next"
        >
          <span class="carousel-control-next-icon" aria-hidden="true"></span>
          <span class="sr-only">Next</span>
        </a>

        <img :src="getImgUrl(currentPokemon.id)" />
        <h2>{{currentPokemon.name}}</h2>
      </div>
      <span v-for="type in pokemonTypes" v-bind:key="type">
        <div style="display: inline-block">
          <img :src="getTypeImgUrl(type)" height="50px" />
          <p>{{type}}</p>
        </div>
      </span>

      <div v-if="!(!currentPokemon.candy && !currentPokemon.buddyDistance && !currentPokemon.cps)">
        <div class="container card p-2">
          <div class="row">
            <div v-if="currentPokemon.candy" class="pokeCandyBorder col-sm">
              <h4>Candy Amount</h4>
              <div v-for="candy in currentPokemon.candy" v-bind:key="candy.form">
                <span v-if="candy.form">{{candy.form}}: {{candy.candy_required}}</span>
                <span v-else>{{candy.candy_required}}</span>
              </div>
            </div>

            <div v-if="currentPokemon.buddyDistance" class="col-sm">
              <h4>Budy Walk Distance</h4>
              <span>{{currentPokemon.buddyDistance[0].distance}}km</span>
            </div>
          </div>
          <div class="row">
            <div v-if="currentPokemon.cps" class="pokeCpsBorder col-sm">
              <h4>Max CP</h4>
              <div v-for="cp in currentPokemon.cps" v-bind:key="cp.form">
                <span v-if="cp.form">{{cp.form}}: {{cp.max_cp}}</span>
                <span v-else>{{cp.max_cp}}</span>

                <b-progress :value="cp.max_cp" :max="4187" class="mb-3"></b-progress>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

<div class="container card p-2">
          <div class="row">
    <div class="mt-4" style="text-align: left">
      <b-card title="Stats">
       <div v-if="currentPokemon.stats">
         <div>Base Attack: {{currentPokemon.stats[0].base_attack}}</div>
         <div>Base Defense: {{currentPokemon.stats[0].base_defense}}</div>
         <div>Base Stamina: {{currentPokemon.stats[0].base_stamina}}</div>
         </div>
      </b-card>
    </div>
          </div>
</div>
  </div>
</template>

<script>
export default {
  name: "Details",
  props: ["id", "pokemon"],
  watch: {
    "$route.params.id": function() {
      this.getPokemon();
    }
  },
  created() {
    this.getPokemon();
  },
  data: function() {
    return {
      mobileScreen: window.innerWidth < 576,
      currentPokemon: null,
      pokemonTypes: null
    };
  },
  methods: {
    getPokemon() {
      const currentPokemon = JSON.parse(localStorage.getItem("allPokemon"))[
        this.$route.params.id
      ];
      const types = [];

      if (currentPokemon["types"]) {
        currentPokemon.types.forEach(type => {
          type.type.forEach(pokemonType => {
            if (types.indexOf(pokemonType) === -1) {
              types.push(pokemonType);
            }
          });
        });
      }
      this.currentPokemon = currentPokemon;
      this.pokemonTypes = types;
    },
    updatePokemon(id, action) {
      const pokemonObj = JSON.parse(localStorage.getItem("allPokemon"));
      let nextPokemonId = id;
      if (action === "next") {
        nextPokemonId++;
        while (!pokemonObj[nextPokemonId]) {
          nextPokemonId++;
        }
      } else if (action === "previous") {
        nextPokemonId--;
        while (!pokemonObj[nextPokemonId]) {
          nextPokemonId--;
        }
      }
      this.$router.push({
        path: `${nextPokemonId}`
      });
    },
    getImgUrl(pokemon_id) {
      let id = ("000" + pokemon_id).slice(-3);
      if ([412].indexOf(id) !== -1)
        return require("./../assets/pokemon_icons/pokemon_icon_" +
          id +
          "_11.png");
      return require("./../assets/pokemon_icons/pokemon_icon_" +
        id +
        "_00.png");
    },
    getTypeImgUrl(type) {
      return require("./../assets/pokemon_types/" +
        type.toLowerCase() +
        ".png");
    },
    getWidth(cp) {
      console.log(cp);
    },
    goHome() {
      this.$router.push("/");
    }
  }
};
</script>

<style scoped>
.container {
  text-align: center;
}
.backHome {
  background-color: rgba(80, 80, 80, 0.5);
  border-radius: 50%;
  height: 50px;
  position: fixed;
  left: 10px;
  width: 50px;
  z-index: 3;
}
.backHome:hover {
  cursor: pointer;
  background-color: rgba(80, 80, 80, 0.8);
}
.center {
  position: relative;
  top: 10px;
  text-align: center;
  vertical-align: middle;
}
span img {
  margin: 0.2rem;
}
</style>
