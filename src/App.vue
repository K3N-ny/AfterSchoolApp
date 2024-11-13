<template>
 <div id="app">
  <div class="container">
    <button v-on:click="showCheckout" :disabled="isCartEmpty"> {{ cartCount }} checkout</button>
    <br>
    <br>
    <input type="search" placeholder="Search here"><input type="button"  value="Search">

    <h3>VIEW</h3>
    <br>
    <label>Ascending</label><input type="radio" v-model="order" value="ascending"> <label>Descending</label> <input  type="radio" v-model="order" value="descending">
    <br>
    <h3> SORT</h3><br>
    <label>Subject</label><input type="radio" v-model="sort" value="subject">
    <label>Location</label><input type="radio" v-model="sort" value="location">
    <label>Price</label><input type="radio" v-model="sort" value="price">
    <label>Space</label><input type="radio" v-model="sort" value="space">


    <div v-if="showProduct">
       <div v-for="products in sortProducts">
      <!-- product information gotten from the products.js -->

      <img :src="products.images" alt="products.name" class="image"/>
      <p> Subject: {{ products.subject }}</p>
      <p> Location: {{ products.location }}</p>
      <p> Price: £{{ products.price }}</p>
      <p> Spaces: {{ products.spaces }}</p>
      <!-- add to cart -->
     
      <button v-on:click="add(products)" v-if="canAdd(products)"> Add to cart</button>
      <button disabled='disabled' v-else>
        Sold Out </button>
      </div>
      
    </div>

    <div v-else> 
      
      <form>
        <h2> CHECKOUT PAGE</h2>
        <label>First name:</label>
        <input type="text" v-model="firstName" required>
        <label>Last name:</label>
        <input type="text" v-model="lastName" required>
        <label>Address:</label>
        <input type="text" v-model="address" required>
        <label>Phone number:</label>
        <input type="text" v-model="phoneNumber" required>
        <label>Email:</label>
        <input type="email" v-model="email" required>
      </form>
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
      cart:[],
      showProduct:true,
      firstName: '',
      lastName: '',
      address: '',
      phoneNumber: '',
      email: '',
      sort: 'price', 
      order: 'ascending'
    };
  },
  
  methods:{
  // pushing the products into cart array
     add(products){
     this.cart.push(products.id),  
      products.spaces --;
    
    },

    // showCheckout() {
    //   this.showProduct = !this.isCartEmpty;  // Only show checkout if cart is not empty
    // }
    // 

    // },
    showCheckout(){
      if (this.cart.length > 0) {
        this.showProduct = false;
      }
    }
  },
  computed: {
    // display the number of items in the cart
    cartCount() {
      return this.cart.length;
      
 },
  isCartEmpty() {
      return this.cart.length === 0; // Check if cart is empty, then disables the button
    },
  // if the spaces left are zero
  canAdd(){
    // return this.products.spaces > this.cartCount;
    return (products) => {
        // Count how many times the product is in the cart
        const countInCart = this.cart.filter(id => id === products.id).length;
        return countInCart < products.spaces;
  }
   
},
sortProducts(){
  return [...this.products].sort((a, b) => {
    let result = 0;
    if (a[this.sort] < b[this.sort]) result =-1;
    if (a[this.sort] > b[this.sort]) result = 1;
  
  
  return this.order === 'descending' ? -result : result;
 });
}
 }}
</script>
<style>
.image{
  width: 30%;
  height:auto;
  border: solid 2px;
}

</style>