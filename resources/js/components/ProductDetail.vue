<template>
    <div v-if="product" class="min-h-screen p-8 bg-base-300">
        <div class="navbar bg-base-300">
            <div class="flex-1">
                <a @click="$router.back()" class="btn btn-circle btn-ghost">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                    </svg>
                </a>
                <h1 class="text-3xl font-bold">Product Details</h1>
            </div>
        </div>
        
        <div class="grid grid-cols-1 gap-8 mt-4 md:grid-cols-3">
            <div class="flex flex-col space-y-4 md:col-span-2">
                <div class="flex justify-center max-w-full p-4 rounded-md shadow lg:max-w-screen">
                    <img class="w-auto h-auto md:h-[500px] object-cover bg-center" :src="'/storage/' + product.img_path" :alt="product.name">
                </div>
                
                <div class="p-4 rounded-md shadow">
                    <h1 class="mb-4 text-2xl font-bold">{{ product.name }}</h1>
        
                    <div class="flex items-center gap-2 mb-4">
                        <span class="badge badge-primary">{{ product.product_detail?.stock ?? 'N/A' }} in stock</span>
                        <span class="badge badge-secondary">{{ product.product_detail?.sold ?? 0 }} Sold</span>
                    </div>
        
                    <div class="mb-4">
                        <span class="text-xl">${{ formatPrice(product.product_detail?.price) }}</span>
                    </div>
        
                    <div class="mb-4">
                        <p>{{ product.product_detail?.description ?? 'No description available' }}</p>
                    </div>
                </div>
            </div>                

            <div class="w-full p-6 shadow-xl card bg-base-100">
                <h2 class="text-2xl font-semibold">Buy Product</h2>
                <div class="mt-6 space-y-4">
                    <div class="flex items-center space-x-2">
                        <button class="btn btn-square btn-sm" @click="decrementQuantity">-</button>
                        <input type="number" v-model.number="quantity" class="w-16 input input-bordered" min="1" :max="product.product_detail?.stock" />
                        <button class="btn btn-square btn-sm" @click="incrementQuantity">+</button>
                    </div>
                    <button class="w-full btn btn-primary" @click="addToCart">Add to Cart</button>
                    <button class="w-full btn btn-secondary" @click="buyNow">Buy Now</button>
                </div>
            </div>
        </div>
        <Popup v-model:show="popupShow" :title="popupTitle" :message="popupMessage" :type="popupType" />
    </div>
    <div v-else class="flex items-center justify-center min-h-screen">
        <p class="text-xl">Loading...</p>
    </div>
</template>
    
<script>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'
import Popup from './Popup.vue'

export default {
    components: { Popup },
    setup() {
        const route = useRoute()
        const router = useRouter()
        const product = ref(null)
        const quantity = ref(1)
        const popupShow = ref(false)
        const popupTitle = ref('')
        const popupMessage = ref('')
        const popupType = ref('info')

        const showPopup = (title, message, type = 'info') => {
            popupTitle.value = title
            popupMessage.value = message
            popupType.value = type
            popupShow.value = true
        }

        const fetchProduct = async () => {
            try {
                const response = await axios.get(`/product/${route.params.id}`, { withCredentials: true })
                if (response.data) {
                    product.value = response.data
                } else {
                    router.push('/search')
                }
            } catch (error) {
                router.push('/search')
            }
        }

        const formatPrice = (price) => {
            return new Intl.NumberFormat('en-US', { minimumFractionDigits: 0 }).format(price || 0)
        }

        const validateQuantity = (qty, stock) => {
            const parsedQty = parseInt(qty);
            if (isNaN(parsedQty) || parsedQty < 1) {
                showPopup('Error', 'Quantity must be at least 1', 'error');
                return false;
            }
            if (parsedQty > stock) {
                showPopup('Error', 'Quantity cannot exceed available stock', 'error');
                return false;
            }
            return true;
        };

        const incrementQuantity = () => {
            const newQty = quantity.value + 1;
            if (validateQuantity(newQty, product.value.product_detail?.stock)) {
                quantity.value = newQty;
            }
        }

        const decrementQuantity = () => {
            const newQty = quantity.value - 1;
            if (validateQuantity(newQty, product.value.product_detail?.stock)) {
                quantity.value = newQty;
            }
        }

        const addToCart = async () => {
            try {
                if (!validateQuantity(quantity.value, product.value.product_detail?.stock)) return;

                await axios.post('/cart', {
                    product_id: product.value.id,
                    quantity: parseInt(quantity.value)
                }, { 
                    withCredentials: true
                });
                showPopup('Success', 'Product added to cart successfully!', 'success')
                window.dispatchEvent(new Event('cart-updated'));
            } catch (error) {
                showPopup('Error', error.response?.data?.message || 'Failed to add product to cart', 'error')
            }
        }

        const buyNow = () => {
            addToCart()
            router.push('/cart')
        }

        onMounted(() => {
            fetchProduct();
        })

        return {
            product,
            quantity,
            formatPrice,
            incrementQuantity,
            decrementQuantity,
            popupShow,
            popupTitle,
            popupMessage,
            popupType,
            addToCart,
            buyNow
        }
    }
}
</script>