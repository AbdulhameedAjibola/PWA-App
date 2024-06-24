<template>
    <div class="show-lesson">
        <input type="text" placeholder="Search Lessons" class="search-bar" v-model="searchTerm">
        <section class="sort-section">
            <h2>Apply Filters</h2>
            <div class="sort">
                <span class="criteria">
                    <select v-model="sortBy" @change="sortLessons">
                        <option value="">Select Sorting Criteria</option>
                        <option value="subject">By Subject</option>
                        <option value="location">By Location</option>
                        <option value="price">By Price</option>
                        <option value="availability">By Availability</option>
                    </select>
                </span>
                <span class="criteria">
                    <select v-model="sortOrder" @change="sortLessons">
                        <option value="ascending">Ascending Order</option>
                        <option value="descending">Descending Order</option>
                    </select>
                </span>
            </div>
        </section>
        
        <section class="main">
            <div class="lessons" v-for="lesson in localSortedLessons" :key="lesson._id">
                <div class="lesson">
                    <div class="image-holder">
                        <img v-bind:src="getImageUrl(lesson.image)" class="lesson-image" alt="lesson image">
                    </div>
                    <div class="lesson-description">
                        <h2>{{ lesson.subject }}</h2>
                        <p>{{ lesson.description }}</p>
                        <p>Location: {{ lesson.location }}</p>
                        <div class="price-stock">
                            <p> DHS {{ lesson.price }}</p>
                            <span>Available Stock: {{ lesson.availability - cartCount(lesson._id) }}</span>
                        </div>
                        <button class="add-to-cart-button" v-on:click="emitAddToCart(lesson._id)" :disabled="!canAddToCart(lesson)">Add to Cart</button>
                        <div class="message">
                            <span v-if="lesson.availability === cartCount(lesson._id)">Lesson Out of Stock</span>
                            <span v-else-if="lesson.availability - cartCount(lesson._id)">
                                Hurry now!. {{ lesson.availability - cartCount(lesson._id) }} Spaces Left!
                            </span>
                            <span v-else>Seats Available! Buy Now!</span>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </div>
</template>

<script>
import _ from 'lodash';
export default {
    props: ['sortedLessons'],
    data() {
        return {
            sortBy: '',
            serverUrl: 'https://afterschool-cw2.onrender.com',
            searchTerm: '',
            sortOrder: 'ascending',
            localSortedLessons: [...this.sortedLessons],  // Create a local copy of sortedLessons
        };
    },
    watch: {
        sortedLessons: {
            handler(newVal) {
                this.localSortedLessons = [...newVal];
                this.sortLessons();
            },
            immediate: true,
        },
        sortBy() {
            this.sortLessons();
        },
        sortOrder() {
            this.sortLessons();
        },
        searchTerm: 'debouncedSearchLessons',
    },
    methods: {
        async searchLessons() {
      try {
       
        const queries = new URLSearchParams();
        queries.append('subject', this.searchTerm);
        queries.append('location', this.searchTerm);
        const response = await fetch(`${this.serverUrl}/lessons/search?${queries.toString()}`);
        const lessons = await response.json();

        this.localSortedLessons = lessons;
      } catch (error) {
        console.error('Error searching lessons:', error);
      }
    },
    debouncedSearchLessons: _.debounce(function () {
            this.searchLessons();
          }, 200), //
        getImageUrl(imagePath) {
            return `${this.serverUrl}/${imagePath}`;
        },
        sortLessons() {
            let sortedArray = [...this.localSortedLessons];
            if (!this.sortBy) {
                this.localSortedLessons = [...this.sortedLessons];
                return;
            }
            
            if (this.sortBy) {
                if (this.sortBy === 'subject') {
                    sortedArray.sort((a, b) => this.sortFunction(a.subject, b.subject));
                } else if (this.sortBy === 'price') {
                    sortedArray.sort((a, b) => this.sortFunction(a.price, b.price));
                } else if (this.sortBy === 'location') {
                    sortedArray.sort((a, b) => this.sortFunction(a.location, b.location));
                } else if (this.sortBy === 'availability') {
                    sortedArray.sort((a, b) => this.sortFunction(a.availability, b.availability));
                }

                // Reverse the sorted array if descending is selected
                if (this.sortOrder === 'descending') {
                    sortedArray.reverse();
                }
            }

            this.localSortedLessons = sortedArray;
        },
        sortFunction(a, b) {
            if (typeof a === 'string' && typeof b === 'string') {
                return a.localeCompare(b);
            } else {
                return a - b;
            }
        },
        canAddToCart(lesson) {
            return lesson.availability > this.cartCount(lesson._id);
        },
        cartCount(lessonId) {
        return this.$parent.cart.filter(_id => _id === lessonId).length;
    },
        emitAddToCart(lessonId) {
        this.$emit('addToCart', lessonId);
    },
       
    },
}
</script>

<style >
.main {
    margin: 20px 5px 0px 5px;
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}
.image-holder{
    z-index: -1;
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
.lesson {
    
    margin: 5px;
    width: 100%;
    max-width: 350px;
    display: flex;
    align-items: center;
    flex-direction: column;
    padding: 15px;
    border: 3px solid black;
    border-radius: 8px;
}

.lesson-image {
    
    height: 250px;
    width: 100%;
    object-fit: cover;
    border-radius: 8px;
    transition: filter 1s ease-in;
}

.lesson-image:hover {
    filter: grayscale(0%);
}

.lesson-description h2 {
    margin-bottom: 10px;
    color: wheat;
    font-size: 2em;
    font-weight: 700;
}

.price-stock p {
    font-weight: 900;
    font-size: 1.5em;
}

.price-stock span {
    font-size: medium;
}

.add-to-cart-button {
    background-color: wheat;
    padding: 10px;
    font-size: 16px;
    font-weight: 700;
    color: grey;
    border: 1px solid black;
    border-radius: 8px;
    transition: transform 0.3s;
    cursor: pointer;
    margin: 8px;
}

.add-to-cart-button:hover {
    transform: scale(1.08);
}

</style>
