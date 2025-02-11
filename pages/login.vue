<template>
    <div class="flex justify-center items-center min-h-screen bg-base-100">
        <div class="w-full max-w-md p-8 space-y-4 bg-base-100 shadow-md rounded-lg">
            <div class="text-center flex">
                <NuxtLink to="/" class="mx-auto">
                    <NuxtImg src="/icon1.png" alt="Icon"
                        class="drop-shadow-[0_0_1px_rgba(255,255,255,1)] w-[250px] h-[50px] object-cover rounded-lg" />
                </NuxtLink>
            </div>

            <h2 class="text-2xl font-bold text-center">Login</h2>

            <form @submit.prevent="handleLogin">
                <div class="form-control">
                    <label class="label">
                        <span class="label-text">Email</span>
                    </label>
                    <input v-model="form.email" type="email" placeholder="example@email.com"
                        class="input input-bordered" required />
                </div>

                <div class="form-control">
                    <label class="label">
                        <span class="label-text">Password</span>
                    </label>
                    <input v-model="form.password" type="password" placeholder="********" class="input input-bordered"
                        required />
                </div>

                <div class="form-control mt-6">
                    <button class="btn btn-primary">Login</button>
                </div>
            </form>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { toast } from 'vue-sonner'
import { useNuxtApp } from '#app'
import { useRouter } from 'vue-router'
import { useUserStore } from "~/stores/user";
import { useLoadingStore } from "~/stores/loading";

const userStore = useUserStore()
const loadingStore = useLoadingStore()

const form = ref({
    email: '',
    password: ''
})

const { $axios } = useNuxtApp()
const router = useRouter()

// Handle login
const handleLogin = async () => {
    try {
        // Start loading with a custom message
        loadingStore.startLoading('Logging in, please wait...');

        // Make an API request to login with email and password
        const response = await $axios.post('/login', {
            email: form.value.email,
            password: form.value.password
        });

        // Extract the token from the API response
        const userToken = response.data.data.access_token;
        const userEmail = form.value.email;

        // Store token and email in cookies for 7 days
        setCookie('token', userToken, 7);
        setCookie('userEmail', userEmail, 7);

        // Update user data in global state
        userStore.setUserData(userToken, userEmail);

        // Show a success toast message
        toast.success(`Login successful, Welcome!`);

        // Redirect to the home page
        await router.push('/');

    } catch (error) {
        // Get the error message from the API response or default to 'Login failed!'
        const errorMessage = error?.response?.data?.error || 'Login failed!';
        const errorCode = error?.response?.status;
        // console.log(error);

        // Show an error toast message
        toast.error(`Error: ${errorCode} ${errorMessage}`);

    } finally {
        // Always stop the loading overlay, whether login is successful or not
        loadingStore.stopLoading();
    }
}

definePageMeta({
    middleware: 'guest'
});

useHead({
    title: 'Login - Film Hunt',
    meta: [
        { name: 'description', content: 'Explore a vast collection of movies, discover new favorites, and keep track of what you want to watch next with Film Hunt.' },
        { property: 'og:image', content: '/icon3.png' },
        { property: 'og:title', content: 'Discover Your Favorite Movies' },
        { property: 'og:description', content: 'Explore movies.' },
        { name: 'twitter:card', content: 'summary_large_image' },
        { name: 'twitter:image', content: '/icon3.png' },
    ],
});

</script>