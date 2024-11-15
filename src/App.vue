<template>
 <div id="app">
  <div class="container">
    <button v-on:click="showCart" :disabled="isCartEmpty"> {{ cartCount }} cart</button>
    <br>
    <br>
    <input type="search" placeholder="Search here"><input type="button"  value="Search">

    
    
    <div v-if="showProduct">
      <h3>VIEW</h3>
      <br>
      <label>Ascending</label><input type="radio" v-model="order" value="ascending"> <label>Descending</label> <input  type="radio" v-model="order" value="descending">
      <br>
      <h3> SORT</h3><br>
      <label>Subject</label><input type="radio" v-model="sort" value="subject">
      <label>Location</label><input type="radio" v-model="sort" value="location">
      <label>Price</label><input type="radio" v-model="sort" value="price">
      <label>Space</label><input type="radio" v-model="sort" value="space">
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
    
    <div v-else >
    <h2>Cart</h2> 
      <div v-for="(products, index) in cart" :key="index">
        <h3>{{ products.subject }}</h3>
        <p>{{ products.price }}</p>
        <button @click="deleteCart(index)"> X </button>

      </div>
      <form @submit.prevent="submitForm">
        <h2> BILLING ADDRESS</h2>
        <label>Name:</label>
        <input type="text" v-model="Name"><br>
        <span v-if="nameError" class="error">{{ nameError }}</span><br>
        <label>Phone number:</label>
        <input type="text" v-model="phoneNumber"><br>
        <span v-if="phoneError" class="error">{{ phoneError }}</span><br>
        <button type="submit" :disabled="!isCheckoutEnabled">Checkout</button>
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
      Name: '',
      phoneNumber: '',
      sort: 'price', 
      order: 'ascending',
      nameError: '',
      phoneError: '',
    };
  },
  
  methods:{
  // pushing the products into cart array
     add(products){
     this.cart.push(products),  
      products.spaces --;
    
    },
    
         
      //cart button only enabled when there is one or more products in it
      showCart(){
        if (this.cart.length > 0) {
          this.showProduct = !this.showProduct;

        }
      },
      deleteCart(index) {
        const removedProduct = this.cart.splice(index, 1)[0];
      const originalProduct = this.products.find((p) => p.id === removedProduct.id);
      if (originalProduct) originalProduct.spaces++; // Restore the spaces for the removed product
      },
      submitForm(){
        
      // Reset errors
      this.nameError = '';
      this.phoneError = '';
      
      const namePattern = /^[A-Za-z\s]+$/;  // Allows letters and spaces
      const phonePattern = /^[0-9]+$/;     // Allows only numbers

      if (!namePattern.test(this.Name.trim())) {
        this.nameError = 'Name must contain letters only.';
      }

      if (!phonePattern.test(this.phoneNumber.trim())) {
        this.phoneError = 'Phone number must contain numbers only.';
      }

      if (!this.nameError && !this.phoneError) {
        // If no errors, proceed with form submission
        console.log("Billing Information:", {
          Name: this.Name,
          phoneNumber: this.phoneNumber,
      });
      this.cart = [];
      alert('Order has been submitted');
        this.Name = "";
        this.phoneNumber = "";
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
isCheckoutEnabled() {
 return ( this.Name.trim() !== "" && this.phoneNumber.trim() !== "")
  

 },
sortProducts(){
  return [...this.products].sort((a, b) => {
    let result = 0;
    if (a[this.sort] < b[this.sort]) result =-1;
    if (a[this.sort] > b[this.sort]) result = 1;
  
  
  return this.order === 'descending' ? -result : result;
 });
}, 

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