---
layout: post
title: Optimistic UI pattern in Vue
---

Optimistic UI is a way of implementing UI in the hope and confidence (optimism) that the server will respond successfully to our API call. With this mental mode in mind, we change the state of our UI confidently before we get a response from our API.

Here is a visual example of both pessimistic UI and an optimistic UI.

![OvsP](https://scontent.fktm8-1.fna.fbcdn.net/v/t1.0-9/117444360_2531181637193391_8272829493525857125_n.jpg?_nc_cat=104&_nc_sid=8024bb&_nc_ohc=qnlLaf09GHoAX_iK3G0&_nc_ht=scontent.fktm8-1.fna&oh=961767f8ba69812ca1dc9ae2188b7f2f&oe=5F5E3152)

I preferred the pessimistic pattern for quite a time as it was more closer to truth and the UI flow was similar to how the system actually worked. On top of that (for better or worse), I would get rid of the loading spinners in the pessimistic UI. My reason being that spinners makes the site look slow. The hope would be that the server would respond fast enough and the change from ‘Mark as complete’ button to the ‘completed’ label would be seamless.

Unfortunately, the server took a few seconds or more :( to respond.It’s always a pain to wait (with or without the loader). As Users expect to get a response for their action in less than a second(or half) and everyone loves a really fast site, I have begun to shift my preference from pessimistic UI to an optimistic one.
 
Enough background talk. Below, I will show some code patterns I wrote for my projects to turn the pessimistic UI into an optimistic one. I have used a temporary variable to store the optimistic state of our UI (`tempIsCompleted`, `tempIsProdutInCart` is the following examples respectively).

    <template>
      <div>
        <span v-if="isCompleted">Completed</span>
        <button v-else @click="markAsComplete">Mark as Complete</button>
      </div>
    </template>
    
The Script:

    <script>
     export default {
      data(){
        return {
          apiResponse: '',
          tempIsCompleted: false
        }
      },
      computed: {
        isCompleted() {
          return tempIsCompleted || apiResponse.is_completed
        }
      },
      methods: {
        getApiResponse(){
          axios.get('url').then(( {data} )=> {
            this.apiResponse = data
          })
        },
        markAsComplete() {
          this.tempIsCompleted = true // leap of Optimism
          axios.patch('url', { is_completed : true}).then(({data}) => {

            // already expected data.is_completed : true

          }).catch((err) => {
            this.tempIsCompleted = false // error handling: reverting back the updated State

          })
        }
      }
    }
    </script>

Here's another example from an E-commerce site.

    <template>
      <div>
        <h1> {{ product.name }}</h1>
        <span v-if="finalIsProductInCart"> Added to Cart</span>
        <button v-else @click="addProductToCart"></buttotn>
      </div>
    </template>

    <script>
      import { mapState, mapActions } from "vuex";
      export default {
        props: ['product'],
        data() {
          return {
            tempIsProdutInCart: false
          }
        },
        computed: {
          ...mapState("cart", ["carts"]),
          isProductInCart() {
            if (this.carts && this.carts.length) {
              return this.carts.find((item) => {
                  return item.id == this.product.id;
                });
            }
          },
          finalIsProductInCart(){
          return this.tempIsProductInCart || this.isProductInCart 
          }
        },
        watch: {
          isProductInCart() {
            this.tempIsProductInCart = false 
            // Once the cart is updated with actual data, 
            // let's use that instead of the temporary state
          }
        },
        methods: {
          ...mapActions("cart", ["addToCart"]),
          addProductToCart() {
            try {
              this.tempIsProductInCart = true
              this.addToCart(this.product);
            } catch (err) {
                this.tempIsProductInCart = false
            }
          }
        }  
      }
    </script>
    
It feels like a cunning trick (that's why I stuck to Pessimistic UI for a while). However it's not a dark pattern as one might think. We just have enough confidence in our system to return the right response and with this confidence we can act beforehand and give our users a better experience.

