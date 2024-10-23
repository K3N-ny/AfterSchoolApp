<template>
 <div id="app">
  <div class="container">
    <input type="button" value="checkout" ><br><br>
    <input type="search" placeholder="Search here"><input type="button"  value="Search">
       <div v-for="products in products">
      <!-- product information gotten from the products.js -->

      <img :src="products.images" alt="products.name" class="image"/>
      <p> Subject: {{ products.subject }}</p>
      <p> Location: {{ products.location }}</p>
      <p> Price: £{{ products.price }}</p>
      <p> Spaces: {{ products.spaces }}</p>
      <!-- add to cart -->
      <input type="button" value="Add to cart" v-on:click="add">
    </div>

    </div>
  </div>
 
 
</template>

<script>
import products from '@/assets/products.js';


 export default {
    data() {
      return {
      products: products,  
          
    }
  },
  cart:[],
  methods:{
    add:function(){
      this.cart.push(this.product.id);
    }
   
  }
}
</script>
<style>
.image{
  width: 30%;
  height:auto;
  border: solid 2px;
}

</style>