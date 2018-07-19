<template>
  <section class="container">
    <div>
      <h1 class="mt-2">Checkout</h1>
      <div class="row">
        <div class="col-md-6 offset-md-3 mt-3">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Cart</h5>
              <p class="card-text">{{ numberOfItems }} items selected:</p>
              <ul>
                <li v-for="dish in selectedDishes" :key="dish.id" class="card-text mb-2">
                  Name: {{ dish.name }} (${{ dish.price }}) ({{ dish.quantity }})
                  <button class="btn btn-sm btn-success" @click="addToCart(dish)">+</button>
                  <button class="btn btn-sm btn-warning ml-2" @click="removeFromCart(dish)">-</button>
                </li>
              </ul>
              <h5 class="card-text">
                Total: ${{ price }}
              </h5>
              <p v-if="!selectedDishes.length">Please select some items.</p>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6 offset-md-3 mt-3">
          <form autocomplete="off" @submit.stop.prevent="handleSubmit">
            <div class="form-group">
              <label for="address">Address</label>
              <b-form-input
                id="address"
                v-model="address"
                type="text"
                autofocus="true"
                placeholder="Enter your address"
                required/>
            </div>
            <div class="form-group">
              <label for="city">City</label>
              <b-form-input
                id="city"
                v-model="city"
                type="text"
                autofocus="true"
                placeholder="Enter your city"
                required/>
            </div>
            <div class="form-group">
              <label for="state">State</label>
              <b-form-input
                id="state"
                v-model="state"
                type="text"
                autofocus="true"
                placeholder="Enter your state"
                required/>
            </div>
            <div class="form-group">
              <label for="zipCode">Postal Code</label>
              <b-form-input
              id="zipCode"
              v-model="zipCode"
              type="text"
              autofocus="true"
              placeholder="Enter your postal code"
              required/>
            </div>
            <div class="form-group">
              <label for="card">Credit Card</label>
              <card
                class="form-control"
                stripe="pk_test_Sg6kbAKhEK3apFB3kjF3qqMO"
              />
            </div>
            <button :disabled="loading" type="submit" class="btn btn-primary btn-block mt-3">Submit</button>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { Card, createToken } from 'vue-stripe-elements-plus'
import { mapMutations } from 'vuex'
import Strapi from 'strapi-sdk-javascript/build/main'
const apiUrl = process.env.API_URL || 'http://localhost:1337'
const strapi = new Strapi(apiUrl)
export default {
  components: {
    Card
  },
  data() {
    return {
      address: '',
      city: '',
      state: '',
      zipCode: '',
      loading: false
    }
  },
  computed: {
    id() {
      return this.$route.params.id
    },
    dishes() {
      return this.$store.getters['dishes/list']
    },
    selectedDishes() {
      return this.$store.getters['cart/items']
    },
    price() {
      return this.$store.getters['cart/price']
    },
    numberOfItems() {
      return this.$store.getters['cart/numberOfItems']
    }
  },
  methods: {
    async handleSubmit() {
      this.loading = true
      let token
      try {
        const response = await createToken()
        token = response.token.id
      } catch (err) {
        alert('An error occurred.')
        this.loading = false
        return
      }
      try {
        await strapi.createEntry('order', {
          amount: this.$store.getters['cart/price'],
          dishes: this.$store.getters['cart/items'],
          address: this.address,
          zipCode: this.zipCode,
          city: this.city,
          state: this.state,
          token
        })
        alert('Your order has been successfully submitted.')
        this.emptyCart()
        this.$router.push('/')
      } catch (err) {
        this.loading = false
        alert('An error occurred.')
      }
    },
    ...mapMutations({
      addToCart: 'cart/add',
      removeFromCart: 'cart/remove',
      emptyCart: 'cart/emptyList'
    })
  }
}
</script>
