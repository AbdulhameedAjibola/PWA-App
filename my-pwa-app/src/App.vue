<template>
  <div id="app">
    <header>
      <div class="navbar">
        <div class="logo">
          <h2>{{ header }}</h2>
        </div>
        <ul class="links">
          <li><input type="text" placeholder="Search Lessons" class="search-bar" v-model="searchTerm"></li>
          <li>
            <button class="show-checkout" @click="toggleComponent" v-if="cartItemCount >= 1">View Cart ({{ cartItemCount }})</button>
            <button class="show-checkout" v-else-if="isCartEmpty()">View Cart ({{ cartItemCount }})</button>
             <button class="show-checkout"  v-else disabled>View Cart ({{ cartItemCount }})</button>
          </li>
        </ul>
        <div class="toggle_btn" @click="toggleDropdown">
          <i class="fa-solid fa-bars"></i>
        </div>
      </div>

      <div class="dropdown_menu">
        <input type="text" placeholder="Search Lessons" class="search-bar" v-model="searchTerm">
        <div class="cart-buttons">
          <button class="show-checkout" @click="toggleComponent" v-if="cartItemCount >= 1">View Cart ({{ cartItemCount }})</button>
            <button class="show-checkout" v-else-if="isCartEmpty()">View Cart ({{ cartItemCount }})</button>
             <!-- <button class="show-checkout"  v-else disabled>View Cart ({{ cartItemCount }})</button> -->
        </div>
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
      searchTerm: '',
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
    async searchLessons() {
      try {
        if (!this.searchTerm.trim()) {
          alert('Search term cannot be empty');
          return;
        }

        const queries = new URLSearchParams();
        queries.append('subject', this.searchTerm);
        queries.append('location', this.searchTerm);
        const response = await fetch(`${this.serverUrl}/lessons/search?${queries.toString()}`);
        const lessons = await response.json();

        this.sortedLessons = lessons;
      } catch (error) {
        console.error('Error searching lessons:', error);
      }
    },
    isCartEmpty() {
            
        }, 
    toggleDropdown() {
      const toggleBtnIcon = document.querySelector('.toggle_btn i');
      const dropDownMenu = document.querySelector('.dropdown_menu');
      dropDownMenu.classList.toggle('open');
      const isOpen = dropDownMenu.classList.contains('open');
      toggleBtnIcon.className = isOpen ? 'fa-solid fa-xmark' : 'fa-solid fa-bars';
    }
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
}
.navbar {
  width: 100%;
  height: 60px;
  width: 100vw;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-around;
}
.navbar .links {
  display: flex;
  gap: 4rem;
}
li {
  list-style: none;
}
.navbar .toggle_btn {
  color: black;
  font-size: 1.5rem;
  cursor: pointer;
  display: none;
}
.dropdown_menu {
  display: none;
  position: absolute;
  top: 60px;
  height: 0;
  width: 300px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  border-radius: 10px;
  overflow: hidden;
  transition: height 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.dropdown_menu.open {
  height: 240px;
  width: 400px;
}
.dropdown_menu li {
  padding: 0.7rem;
  display: flex;
  align-items: center;
  justify-content: center;
}
@media (max-width: 800px) {
  .navbar .links,
  .navbar .action_btn {
    display: none;
  }
  .navbar .toggle_btn {
    display: block;
  }
  .dropdown_menu {
    display: block;
  }
}
@media (max-width: 570px) {
  .dropdown_menu {
    left: 2rem;
    width: unset;
  }
}
.search-bar {
  width: 300px;
  padding: 6px;
  margin: 10px;
  font-size: 16px;
  background-color: white;
  border-radius: 8px;
  border: 1px black solid;
  outline: none;
}
</style>
