<template>
 <div id="app">
  <div class="container">
    <button v-on:click="showCart" :disabled="isCartEmpty"> {{ cartCount }} cart</button>
    <br>
    <br>
    <input
        type="text"
        v-model="searchQuery"
        placeholder="Search here"
      />
      <button @click="clearSearch" v-if="searchQuery" >Clear</button>

    
    
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
      <div v-if="sortProducts.length" class="container">
        
      
      <div v-for="products in sortProducts" id="card">
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
          <p>No products match your search.</p>
        </div>
      </div>
    
    <div v-else id="cart-section">
    <h2>Cart</h2> 
      <div v-for="(products, index) in cart" :key="index" class="cart-item">
        <img :src="products.images" alt="products.name" class="image">
        <h3>{{ products.subject }}</h3>
        <p>{{ products.price }}</p>
        <button @click="deleteCart(index)"> X </button>

      </div>
      <form @submit.prevent="submitForm" id="billing-form">
        <h2> BILLING ADDRESS</h2>
        <label>Name:</label>
        <input type="text" v-model="Name"><br>
        <span v-if="nameError" class="error">{{ nameError }}</span><br>
        <label>Phone number:</label>
        <input type="text" v-model="phoneNumber"><br>
        <span v-if="phoneError" class="error">{{ phoneError }}</span><br>
        <button type="submit" :disabled="!isCheckoutEnabled || isSubmitting">{{ isSubmitting ? 'Submitting...' : 'Checkout' }}</button>
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
      searchQuery: ''
    };
  },
  
  methods:{
  // pushing the products into cart array
     add(products){
     this.cart.push(products),  
      products.spaces --;
    
    },
    clearSearch() {
      this.searchQuery = ''; // Reset search query
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
          const order = {
        customer: {
          name: this.Name,
          phone: this.phoneNumber,
        },
        items: this.cart.map(product => ({
          id: product.id,
          subject: product.subject,
          price: product.price,
          quantity: 1, // Assuming quantity is 1 for each product
        })),
        total: this.cart.reduce((sum, product) => sum + product.price, 0),
      };
  
      // Make POST request to backend
      fetch("http://localhost:5000/api/orders", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order),
      })
        .then(response => {
          if (!response.ok) {
            throw new Error("Failed to submit order");
          }
          return response.json();
        })
        .then(data => {
          console.log("Order submitted:", data);
          alert(data.message || "Order placed successfully!");
          
  
          // Clear cart and reset form
          this.cart = [];
          this.Name = "";
          this.phoneNumber = "";
          this.showProduct = true;
        })
        .catch(error => {
          alert("Failed to submit order. Please try again.");
        })
        .finally(() => {
          this.isSubmitting = false; // Stop loading indicator
        }
        );
    
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
  const searchQuery = this.searchQuery.toLowerCase();
      const filtered = this.products.filter((product) => {
        return (
          product.subject.toLowerCase().includes(searchQuery) ||
        product.location.toLowerCase().includes(searchQuery) ||
        product.price.toString().includes(searchQuery) || 
        product.spaces.toString().includes(searchQuery)
        );
      });
      return filtered.sort((a, b) => {
        let result = 0;
        if (a[this.sort] < b[this.sort]) result = -1;
        if (a[this.sort] > b[this.sort]) result = 1;
        return this.order === 'descending' ? -result : result;
      });
//   return [...this.products].sort((a, b) => {
//     let result = 0;
//     if (a[this.sort] < b[this.sort]) result =-1;
//     if (a[this.sort] > b[this.sort]) result = 1;
  
  
//   return this.order === 'descending' ? -result : result;
//  });
}, 

    }
 }
</script>
<style>
 .img{
  width: 30%;
  height:20%;
  
} 
/* #card{
  display: flex;
  border: transparent 5px;
  padding: 2%;
  margin: 0%;
  justify-content: center;
  align-items: center;
  width: 100%; */ 
  /* position: relative; */

  
/* } */
#card {
  display: flex;
  flex-direction: column; /* Stack items vertically */
  align-items: center; /* Horizontally center items */
  justify-content: center; /* Vertically center items if needed */
  padding: 2%;
  margin: 10px auto; /* Add spacing between cards and center the card */
  width: 80%; /* Optional: Adjust width as needed */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Shadow effect */
  border-radius: 8px; /* Rounded corners */
  background-color: #fff; /* Optional: Background color */
}
#card:hover {
  transform: scale(1.02); /* Slightly enlarge the card on hover */
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2); /* Increase shadow on hover */
}


#card img {
  width: 30%; /* Adjust the image width */
  height: 20%; /* Adjust the image height */
  object-fit: cover; /* Keep image proportions */
  border-radius: 4px; /* Rounded corners for the image */
  margin-bottom: 10px; /* Add spacing below the image */
}

#card p {
  margin: 5px 0; /* Adjust spacing between text */
  text-align: center; /* Center-align the text */
}

#card button {
  margin-top: 10px; /* Add space above buttons */
  padding: 10px 15px; /* Button size */
  border: none; /* Remove button border */
  background-color: #007BFF; /* Button color */
  color: #fff; /* Text color */
  border-radius: 4px; /* Rounded corners */
  cursor: pointer; /* Pointer cursor on hover */
  transition: background-color 0.3s ease, transform 0.3s ease; /* Smooth transition effect */
}

#card button:hover {
  background-color: #0056b3; /* Darker shade on hover */
  transform: scale(1.05); /* Slightly increase button size on hover */
}

#card button:disabled {
  background-color: #ccc; /* Disabled button color */
  cursor: not-allowed; /* Indicate unclickable */
}

#card button:disabled:hover {
  transform: none; /* No scaling for disabled buttons */
}
/* General Button Styling */
button {
  padding: 10px 15px; /* Adjust padding */
  border: none; /* Remove default border */
  border-radius: 4px; /* Rounded corners */
  background-color: #007BFF; /* Button background color */
  color: white; /* Button text color */
  font-size: 14px; /* Font size */
  cursor: pointer; /* Pointer cursor on hover */
  transition: background-color 0.3s ease, transform 0.3s ease; /* Smooth hover transition */
}

/* Cart Button */
button[disabled] {
  background-color: #ccc; /* Disabled button background */
  cursor: not-allowed; /* Disabled cursor */
}

button:hover {
  background-color: #0056b3; /* Darker blue on hover */
  transform: scale(1.05); /* Slight growth on hover */
}

/* Search Bar Styling */
input[type="text"] {
  width: 80%; /* Full width of the container */
  padding: 10px; /* Padding inside the input */
  margin: 10px 0; /* Space around input */
  border: 1px solid #ddd; /* Light gray border */
  border-radius: 4px; /* Rounded corners */
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1); /* Subtle shadow */
  font-size: 14px; /* Font size */
  outline: none; /* Remove outline */
  transition: box-shadow 0.3s ease, border-color 0.3s ease; /* Smooth focus transition */
}

input[type="text"]:focus {
  border-color: #007BFF; /* Blue border on focus */
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5); /* Glow effect */
}

/* Radio Buttons for View & Sort */
input[type="radio"] {
  accent-color: #007BFF; /* Customize the radio button color */
  transform: scale(1.2); /* Slightly enlarge radio buttons */
  cursor: pointer; /* Pointer cursor */
  margin-right: 5px; /* Space between radio button and label */
}

label {
  font-size: 14px; /* Consistent label size */
  margin-right: 10px; /* Space between labels */
  color: #333; /* Darker text color for readability */
}

/* Section Headers */
h3 {
  color: #007BFF; /* Matching theme color */
  font-weight: bold; /* Emphasize headers */
  margin-bottom: 10px; /* Space below headers */
}

/* Cart Section Styling */
#cart-section {
  padding: 20px;
  margin: 0 auto;
  width: 90%;
  max-width: 800px; /* Restrict the width for better readability */
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow */
}

/* Cart Items */
#cart-section .cart-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #ddd; /* Separator between items */
  padding: 10px 0;
}

#cart-section .cart-item img {
  width: 30%;
  height: 20%;
  object-fit: cover; /* Maintain proportions */
  border-radius: 4px;
  margin-right: 10px;
  
}

#cart-section .cart-item h3 {
  flex: 1; /* Allow title to take up space */
  margin: 0 15px; /* Space between image and text */
  font-size: 16px;
  color: #333;
}

#cart-section .cart-item p {
  font-size: 14px;
  margin: 0;
  color: #666; /* Lighter text for price */
}

#cart-section .cart-item button {
  background-color: #ff4d4d; /* Delete button in red */
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

#cart-section .cart-item button:hover {
  background-color: #cc0000; /* Darker red on hover */
}

/* Billing Form */
#billing-form {
  margin-top: 20px;
  padding: 20px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow */
}

#billing-form h2 {
  font-size: 20px;
  color: #007BFF; /* Theme color */
  margin-bottom: 20px;
  text-align: center;
}

#billing-form label {
  display: block;
  font-size: 14px;
  color: #333;
  margin-bottom: 5px;
}

#billing-form input {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

#billing-form input:focus {
  border-color: #007BFF;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5); /* Glow effect */
}

#billing-form button {
  display: block;
  width: 100%;
  padding: 10px 0;
  background-color: #007BFF;
  color: white;
  font-size: 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

#billing-form button:hover {
  background-color: #0056b3;
  transform: scale(1.02); /* Slightly enlarge on hover */
}

#billing-form .error {
  color: #ff4d4d; /* Red for error messages */
  font-size: 12px;
  margin-bottom: 10px;
}

/* Cart Page Headers */
h2, h3 {
  color: #007BFF; /* Theme color for headings */
  text-align: center;
  margin-bottom: 15px;
}


</style>