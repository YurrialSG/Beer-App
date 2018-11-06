<template>
  <v-app>
    <v-toolbar
      app
    >
      <v-btn flat fab @click.prevent="aboutPopup = !aboutPopup">
        <v-icon>info</v-icon>
      </v-btn>
      <router-link to="/" tag="v-toolbar-title" style="cursor: pointer;">{{ title }}</router-link>
      <v-spacer></v-spacer>
      <v-btn flat @click.prevent="cartPopup = !cartPopup">
        <v-badge right>
          <span slot="badge" v-if="cartCount">{{ cartCount }}</span>
          <v-icon>shopping_cart</v-icon>
        </v-badge>
      </v-btn>
    </v-toolbar>
    <v-content>
      <router-view/>
      <v-dialog v-model="cartPopup" persistent max-width="500px">
        <v-card>
          <v-card-title>
            <div class="headline">Seu carrinho</div>
          </v-card-title>
          <v-card-text>
            <div class="subheading" v-for="item in cartItems" :key="item.id">
              <v-list-tile>
              <v-btn flat fab small color="error" @click.prevent="toggleCart(item.id)">
                <v-icon>clear</v-icon>
              </v-btn>
              <v-list-tile-avatar>
                <img :src="item.image_url">
              </v-list-tile-avatar>
              {{ item.name }}     
              </v-list-tile> 
            </div>
            <v-alert v-if="!cartItems.length" :value="true" type="warning">
              Nenhum produto inserido
            </v-alert>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn flat @click.prevent="cartPopup = !cartPopup">Fechar</v-btn>
            <v-btn flat @click.prevent="onCartSubmit" class="success">Comprar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="aboutPopup" max-width="400px">
        <v-card>
          <v-card-title>
            <div class="headline">{{ title }}</div>
          </v-card-title>
          <v-card-text class="text-xs-center">
            <div class="subheading">Desenvolvido por <b>Yuri Silveira</b></div>
            <br />
            <div class="body-2">Visualizar código em <a href="https://github.com/YurrialSG" target="_blank">GitHub (@YurrialSG)</a></div>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn flat @click.prevent="aboutPopup = !aboutPopup">Fechar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-snackbar
        :timeout="6000"
        v-model="checkoutPop"
      >
        Opa! Fim da aplicação !!
        <v-btn flat color="pink" @click.native="checkoutPop = false">Fechar</v-btn>
      </v-snackbar>
    </v-content>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      title: "Beer App",
      cartPopup: false,
      aboutPopup: false,
      checkoutPop: false
    };
  },
  name: "App",
  methods: {
    onCartSubmit() {
      this.checkoutPop = true;
    },
    toggleCart(id) {
      this.$store.dispatch("toggleCart", { id });
    }
  },
  computed: {
    cartCount() {
      return this.$store.getters.getCart.length;
    },
    cartItems() {
      const cartIds = this.$store.getters.getCart;
      const allBeers = this.$store.getters.getAllBeers;
      let cart = [];
      for (let id of cartIds) {
        let el = allBeers.filter(b => b.id === id);
        cart.push(el[0]);
      }
      return cart;
    }
  }
};
</script>
