<template>
  <div id="app">
    <header>
     
        <div class="logo">
          <h2>{{ header }}</h2>
        </div>
       <div class="">
        <button class="show-checkout" @click="toggleComponent" v-if="cartItemCount >= 1">View Cart ({{ cartItemCount }})</button>
            
             <button class="show-checkout"  v-else disabled>View Cart ({{ cartItemCount }})</button>
       </div>
      

      
    </header>
    <!-- dynamic component rendering -->
    
        <component 
      :is="activeTab" 
      :sortedLessons="sortedLessons" 
      v-on:addToCart="addToCart" 
      v-if="activeTab === 'LessonComponent'"
    ></component>

    <component :is="activeTab" 
    v-else :cart="cart"
     :lessons="lessons"
     @clearCart="clearCart"
     :toggleComponent="toggleComponent"
      v-on:removeFromCart="removeFromCart">
    </component>
    
  </div>
</template>

<script>
import LessonComponent from "./components/LessonComponent.vue";

import CheckoutComponent from "./components/CheckoutComponent.vue";
export default {
  name: 'App',
  components:{
    LessonComponent,
    CheckoutComponent
  },
  data() {
    return {
      header: "Hameed's Extra lesson's Site",
      checkout: "Checkout Page",
      lessons: [],
      serverUrl: 'https://afterschool-cw2.onrender.com',
      cart:[],
      sortedLessons: [],
      
      activeTab: 'LessonComponent',
      
    };
  },
  methods: {
    addToCart(lessonId) {
        this.cart.push(lessonId);
    },
    toggleComponent() {
      this.activeTab = this.activeTab === 'LessonComponent' ? 'CheckoutComponent' : 'LessonComponent';
    },
    removeFromCart(_id) {
    // Find the index of the item with the given _id
    let index = this.cart.indexOf(_id);
    // If the item is found, remove it from the cart
    if (index !== -1) {
        this.cart.splice(index, 1);
    }
    // Check if the active tab is 'CheckoutComponent' and the cart is empty
    if (this.activeTab === 'CheckoutComponent' && this.cart.length === 0) {
        this.toggleComponent();
    }
},
clearCart() {
      this.cart = [];
    },
    async getLessons() {
      try {
        
        const response = await fetch(`${this.serverUrl}/lessons`);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        this.lessons = await response.json();
        this.sortedLessons = [...this.lessons]; // Initialize sortedLessons with fetched lessons
        this.cart = []; 
      } catch (error) {
        console.error('There was a problem with the fetch operation:', error);
      }
    },
    
   
   
  },
  computed: {
        cartItemCount() {
            return this.cart.length || "";
        },
        
    },
  mounted() {
    this.getLessons();
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: "Rubik", sans-serif;
  font-optical-sizing: auto;
  font-weight: 400;
  font-style: normal;
}
header {
  position: relative;
  padding: 0 2rem;
  display: flex;
  padding: 15px;
  background-color: wheat;
  justify-content: space-around;
  align-items: center;
}
.show-checkout
{
  padding: 15px;
}



</style>
