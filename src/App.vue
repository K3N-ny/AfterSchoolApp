<template>
 <div id="app">
  <div class="container">
    <div class="row">
      <p> Subject: {{ subject }}</p>
      <br>
      <p> Location: {{ location }}</p>
      <br>
      <p> Price: {{ price }}</p>
      <br>
      <p> Spaces: {{ spaces }}</p>
    </div>
  </div>
 </div>
 
</template>

<script>
 export  default {
    data() {
      return {
        subject:  'Maths',
        location: 'London',
        price: 100,
        spaces: 5,
      }
    }
  }
</script>