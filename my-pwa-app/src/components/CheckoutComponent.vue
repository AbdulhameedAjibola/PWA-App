<template>
    <div>
        <p>Welcome to Checkout component</p>
        <div class="showCheckout">
            <h2 class="title">{{ checkout }}</h2>
            <div class="cart-items">
                <div class="cart-item" v-for="lessonID in cartSet" :key="lessonID">
                    <div v-if="lesson = lessons.find(lesson => lesson._id === lessonID)" class="item-data">
                        <div class="item-image">
                            <img :src="getImageUrl(lesson.image)" class="cart-lesson-image" alt="lesson image">
                        </div>
                        <h2>{{ lesson.subject }}</h2>
                        <p>{{ lesson.description }}</p>
                        <p>{{ lesson.location }}</p>
                        <div class="price-stock">
                            <p> DHS {{ lesson.price }}</p>
                            <span>Available Stock: {{ lesson.availability - cartCount(lesson._id) }}</span>
                        </div>
                        <button class="remove-from-cart" @click="emitremoveFromCart(lesson._id)">Remove from Cart</button>
                    </div>
                    
                </div>
            </div>

            <div class="checkout-details">
                <div class="details-input">
                    <h2 class="order-title">Please enter Order Details</h2>
                    <label for="Name">Customer Name</label><br>
                    <input type="text" id="Name" v-model="customerName"><br>
                    <span class="message" v-if="!isValidText()">Please enter your name (Alphabets Only) and not more than 15 characters.</span><br>
                    <label for="Number">Phone Number</label><br>
                    <input type="number" id="Number" v-model="customerNumber"><br>
                    <span class="message" v-if="!isValidNumber()">Please enter your complete Phone Number (Digits Only)</span>
                </div>
                <button v-if="showCheckoutButton()" @click="checkOut" class="place-order">Buy Now!</button>
                <button disabled v-else class="place-order">Buy Now!</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        cart: {
            type: Array,
            required: true
        },
        lessons: {
            type: Array,
            required: true
        },
        toggleComponent: Function
    },
    data() {
        return {
            customerName: '',
            customerNumber: '',
            checkout: "Checkout Page",
            serverUrl: 'https://afterschool-cw2.onrender.com',
            localCart: [...this.cart]
        }
    },
    methods: {
        getImageUrl(imagePath) {
            return `${this.serverUrl}/${imagePath}`;
        },
        cartCount(lessonId) {
        return this.cart.filter(_id => _id === lessonId).length;
    },
    emitremoveFromCart(lessonID){
        this.$emit('removeFromCart', lessonID)
    },
        async checkOut() {
            try {
                const order = {
                    customerName: this.customerName,
                    customerNumber: this.customerNumber,
                    items: this.localCart
                };
                const response = await fetch(`${this.serverUrl}/orders`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(order)
                });

                const data = await response.json();
                if (response.ok) {
                    alert(data.msg);
                    this.updateLessons();
                    this.toggleComponent();
                    this.$emit('clearCart');
                    this.customerName = '';
                    this.customerNumber = '';
                } else {
                    alert('Error placing order');
                }
            } catch (error) {
                console.error('There was a problem with the fetch operation:', error);
                alert('Error placing order');
            }
        },
        showCheckoutButton() {
            return this.isValidNumber() && this.isValidText();
        },
        isValidNumber() {
            return /^[1-9]\d*$/.test(this.customerNumber) && this.customerNumber.length === 11;
        },
        isValidText() {
            return /^[a-zA-Z]+$/.test(this.customerName) && (this.customerName.length >= 3 && this.customerName.length <= 12);
        },
        async updateLessons() {
            try {
                for (const lessonId of this.cartSet) {
                    const lesson = this.lessons.find(lesson => lesson._id === lessonId);
                    if (lesson) {
                        const decrement = this.cartCount(lessonId);
                        const updatedSpaces = lesson.availability - decrement;

                        // Update lesson availability on the server
                        const response = await fetch(`${this.serverUrl}/lessons/${lessonId}`, {
                            method: "PUT",
                            headers: { "Content-Type": "application/json" },
                            body: JSON.stringify({ availability: updatedSpaces })
                        });

                        if (!response.ok) {
                            throw new Error(`Failed to update lesson ${lessonId}`);
                        }

                        // Update lesson availability
                        this.getLessons();
                    }
                }
            } catch (error) {
                console.error('Error updating lessons:', error);
            }
        }
    },
    computed: {
        cartSet() {
            return new Set(this.localCart);
        }
    }
}
</script>

<style>
.showCheckout{
    margin: 30px;
}
.cart-items {
    margin-top: 30px;
    display: flex;
    flex-direction: column;
    gap: 40px;
}

.cart-item {
    
    border: 3px solid black;
    border-radius: 8px;
    
    
    padding: 15px;
    
    gap: 15px;
}

.item-data {
    display: flex;
   gap: 10px;
   align-items: center;
    justify-content: space-around;
}
@media (max-width: 884px) {
  .cart-item
{
    flex-direction: column;
}
.item-data{
    flex-direction: column;
}
}

.cart-lesson-image {
    height: 180px;
    width: auto;

    max-width: 350px;
    border-radius: 8px;
}

.cart-item-description h2 {
    margin-bottom: 10px;
    color: wheat;
    font-size: 2em;
    font-weight: 700;
}

.remove-from-cart {
    background-color: red;
    padding: 10px;
    font-size: 18px;
    height: 50px;
    font-weight: 700;
    color: wheat;
    border: 1px solid black;
    border-radius: 8px;
    transition: opacity 0.3s ease;
    cursor: pointer;
}

.remove-from-cart:active {
    opacity: 0;
}

.checkout-details {
    margin-top: 40px;
}

.details-input h2 {
    text-align: center;
    color: wheat;
}

.details-input input {
    padding: 10px;
    width: 100%;
    max-width: 300px;
    margin: 8px 0;
}

.place-order {
    background-color: wheat;
    padding: 10px;
    font-size: 20px;
    font-weight: 700;
    color: grey;
    border: 1px solid black;
    border-radius: 8px;
    transition: transform 0.5s;
    cursor: pointer;
    margin: 15px;
}

.place-order:hover {
    transform: scale(1.1);
}


</style>
