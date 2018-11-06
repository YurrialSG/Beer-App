<template>
  <v-container>
    <v-layout row wrap>
      <v-flex xs12 md6 class="px-2">
        <v-text-field
          label="Pesquisar por nome"
          v-model="filter.beerName"
        ></v-text-field>
      </v-flex>
      <v-flex xs12 md3 lg4 class="px-2">
        <v-select
          :items="allBeerIbu"
          v-model="filter.beerIbu"
          label="Selecionar"
          single-line
          autocomplete
        ></v-select>
      </v-flex>
      <v-flex xs12 md3 lg2 class="px-2">
        <v-btn block color="info" @click.prevent="onSearch">
          <v-icon left>search</v-icon>Pesquisar
        </v-btn>
      </v-flex>
    </v-layout>
    <v-layout row wrap>
      <v-flex xs12 class="text-xs-right">
        <v-btn flat @click.prevent="onSortByAbv">Ordenar por conteúdo de álcool - ABV
          <v-icon right v-if="sortbyFlag.abv % 3 == 1">arrow_downward</v-icon>
          <v-icon right v-if="sortbyFlag.abv % 3 == 2">arrow_upward</v-icon>
        </v-btn>
      </v-flex>
    </v-layout>
    <v-layout row wrap>
      <v-flex xs12 sm6 md4 class="pa-2" v-for="beer in showBeers" :key="beer.id">
        
        <v-card>

              <v-img
      :src="beer.image_url"
      aspect-ratio="1"
      height="200px"
      contain
    ></v-img>
              <v-card-title primary-title>
      <div>
        <h3 class="headline mb-0">{{beer.name}}</h3>
        <div>{{beer.tagline}}</div>
      </div>
    </v-card-title>
          <v-card-text>
            <v-layout>
              <v-flex xs12 sm8>
                <div class="subheading">{{ beer.style }}</div>
              </v-flex>
              <v-flex xs12 sm4 class="text-xs-left text-sm-right">
                <div class="subheading" v-if="beer.abv">{{ beer.abv }} ABV</div>
                <div class="subheading" v-if="beer.ibu">{{ beer.ibu }} IBU</div>
              </v-flex>
            </v-layout>
          </v-card-text>
        </v-card>



<!-- Adicionando o componente de card -->
  <v-card>
    <!-- Atualizando para melhoria visual -->
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              v-if="!allCart.includes(beer.id)"
              @click.prevent="toggleCart(beer.id)"
              flat color="blue" >Adicionar
            </v-btn>
            <v-btn
              @click.prevent="toggleCart(beer.id)"
              v-else
              flat dark class="orange darken-1">
              <v-icon left>remove</v-icon>Remover do carrinho
            </v-btn>
            <v-btn flat color="orange">Detalhes</v-btn>
          </v-card-actions>
  </v-card>

      </v-flex>
      <v-flex xs12 v-if="noResults" class="px-2">
        <v-alert :value="true" type="error">
          Nenhum resultado encontrado.
        </v-alert>
      </v-flex>
    </v-layout>
    <v-layout>
      <v-flex xs12 class="px-2">
        <v-btn block class="success" @click.prevent="onLoadMore">
          Mostrar mais
        </v-btn>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
export default {
  name: "AppHome",
  data() {
    return {
      showBeers: [],
      filter: {
        beerName: "",
        beerIbu: "Filtrar por IBU"
      },
      tillItem: 6,
      searchFlag: false,
      showSearchedBeers: [],
      sortbyFlag: {
        abv: 0
      }
    };
  },
  computed: {
    allBeerIbu() {
      return ["Filtrar por IBU", ...this.$store.getters.getAllBeerIbu];
    },
    allBeers() {
      return this.$store.getters.getAllBeers;
    },
    noResults() {
      return this.showBeers.length <= 0;
    },
    allCart() {
      return this.$store.getters.getCart;
    }
  },
  methods: {
    onSearch() {
      this.tillItem = 6;
      const beerName = this.filter.beerName.toLowerCase().trim();
      const beerIbu = this.filter.beerIbu.trim();
      let tempAllBeers = this.allBeers;
      let tempShowBeers = [];
      if (beerName != "") {
        tempShowBeers = tempAllBeers = tempAllBeers.filter(
          b => b.name.toLowerCase().indexOf(beerName) !== -1
        );
      }
      if (beerIbu != "Filtrar por IBU") {
        tempShowBeers = tempAllBeers.filter(b => b.ibu.trim() === beerIbu);
      }
      if (beerName == "" && beerIbu == "Filtrar por IBU") {
        tempShowBeers = tempAllBeers;
        this.searchFlag = false;
      } else {
        this.searchFlag = true;
        this.showSearchedBeers = this.sortByAbv(tempShowBeers);
      }
      this.showBeers = tempShowBeers.slice(0, this.tillItem);
    },
    onLoadMore() {
      this.tillItem *= 2;
      if (this.searchFlag)
        this.showBeers = this.showSearchedBeers.slice(0, this.tillItem);
      else this.showBeers = this.allBeers.slice(0, this.tillItem);
    },
    onSortByAbv() {
      let tempShowbeers = [];
      this.sortbyFlag.abv += 1;
      if (this.searchFlag) {
        tempShowbeers = this.showSearchedBeers;
      } else {
        tempShowbeers = this.allBeers;
      }
      tempShowbeers = this.sortByAbv(tempShowbeers);
      this.showBeers = tempShowbeers.slice(0, this.tillItem);
    },
    sortByAbv(obj) {
      const tempAbv = parseInt(this.sortbyFlag.abv);
      if (tempAbv % 3 == 1) {
        return obj.sort(
          (a, b) => (a.abv == b.abv ? 0 : +(a.abv > b.abv) || -1)
        );
      } else if (tempAbv % 3 == 2) {
        return obj.sort(
          (a, b) => (a.abv == b.abv ? 0 : +(a.abv < b.abv) || -1)
        );
      } else {
        return obj.sort((a, b) => (a.id == b.id ? 0 : +(a.id < b.id) || -1));
      }
    },
    dynamicSort(property) {
      var sortOrder = 1;
      if (property[0] === "-") {
        sortOrder = -1;
        property = property.substr(1);
      }
      return function(a, b) {
        var result =
          a[property] < b[property] ? -1 : a[property] > b[property] ? 1 : 0;
        return result * sortOrder;
      };
    },
    toggleCart(id) {
      this.$store.dispatch("toggleCart", { id });
    }
  },
  mounted() {
    this.showBeers = this.allBeers.slice(0, this.tillItem);
  }
};
</script>


<style scoped>
</style>
