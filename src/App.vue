<template>
 <div id="app">
  <div class="container">
    <button> {{ cartCount }} checkout</button><br><br>
    <input type="search" placeholder="Search here"><input type="button"  value="Search">
    <h3>VIEW</h3><br>
    <label>Ascending</label><input type="checkbox" value="ascending"> <label>Descending</label> <input  type="checkbox" value="descending">
    <br>
    <h3> SORT</h3><br>
    <label>Subject</label><input type="checkbox" value="subject">
    <label>Location</label><input type="checkbox" value="location">
    <label>Price</label><input type="checkbox" value="price">
    <label>Space</label><input type="checkbox" value="space">
       <div v-for="products in products">
      <!-- product information gotten from the products.js -->

      <img :src="products.images" alt="products.name" class="image"/>
      <p> Subject: {{ products.subject }}</p>
      <p> Location: {{ products.location }}</p>
      <p> Price: £{{ products.price }}</p>
      <p> Spaces: {{ products.spaces }}</p>
      <!-- add to cart -->
     
      <button v-on:click="add(products)" v-if="canAdd(products)"> Add to cart</button>
      <button disabled='disabled' v-else>
        Add to cart </button>
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
      cart:[ ],   
    };
  },
  
  methods:{
    // pushing the products into cart array
    add(products){
      this.cart.push(products.id);

    }
  },
  computed: {
    // display the number of items in the cart
    cartCount() {
      return this.cart.length;
      
  },
  // if the spaces left are zero
  canAdd(){
    // return this.products.spaces > this.cartCount;
    return (products) => {
        // Count how many times the product is in the cart
        const countInCart = this.cart.filter(id => id === products.id).length;
        return countInCart < products.spaces;
  }
   
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