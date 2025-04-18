<template>
  <div id="app" :style="{ backgroundImage: `url(${backgroundImage})`, backgroundSize: backgroundSize, backgroundPosition: backgroundPosition }">
    <nav class="navbar" v-if="!isAdminPage">
      <!-- Hava durumu bilgisi (sadece şehir sayfalarında göster) -->
      <div v-if="route.path.startsWith('/city')" class="weather-info">
        <span v-if="weather">
          <img :src="`http://openweathermap.org/img/wn/${weather.icon}@2x.png`" alt="Weather Icon" class="weather-icon" />
          {{ weather.description }}: {{ weather.temperature }}°C | Luftfeuchte: {{ weather.humidity }}% | Wind: {{ weather.windSpeed }} m/s
        </span>
        <span v-else>Hava durumu yükleniyor...</span>
      </div>

      <ul class="nav-links">
        <router-link to="/">Home</router-link>

        <!-- Kullanıcı giriş yapmadıysa -->
        <template v-if="!user">
          <router-link to="/login">Log In</router-link>
          <router-link to="/register">Register</router-link>
        </template>

        <!-- Kullanıcı giriş yaptıysa -->
        <div v-if="user" class="user-info">
          <span class="user-icon">👤</span>
          <span class="user-name">{{ user.vorname }}</span>
          <button @click="logout">Log Out</button>
        </div>
      </ul>
    </nav>
    <router-view @user-logged-in="updateUser" @update-background="updateBackground" @update-weather="updateWeather"></router-view>
  </div>
</template>

<script>
import { defineComponent, ref, onMounted, computed } from "vue";
import { useRouter, useRoute } from "vue-router";

// homeW.jpeg resmini import ediyoruz
import HomeWImage from '@/assets/homeW.jpeg';

export default defineComponent({
  name: "App",
  setup() {
    const router = useRouter();
    const route = useRoute();
    const user = ref(null);
    const customBackground = ref(null);
    const weather = ref(null);

    onMounted(() => {
      const storedUser = localStorage.getItem("user");
      if (storedUser) {
        user.value = JSON.parse(storedUser);
      }
    });

    const updateUser = (userData) => {
      user.value = userData;
      localStorage.setItem("user", JSON.stringify(userData));
    };

    const logout = () => {
      localStorage.removeItem("user");
      user.value = null;
      router.push("/login");
    };

    const updateBackground = (image) => {
      customBackground.value = image;
    };

    const updateWeather = (weatherData) => {
      weather.value = weatherData;
    };

    const backgroundImage = computed(() => {
      if (route.path.startsWith("/city") && customBackground.value) {
        return customBackground.value;
      }
      return HomeWImage;
    });

    const backgroundSize = computed(() => {
      if (route.path.startsWith("/city")) {
        return "cover";
      }
      return "100% auto";
    });

    const backgroundPosition = computed(() => {
      if (route.path.startsWith("/city")) {
        return "center";
      }
      return "center top";
    });

    return {
      user,
      logout,
      updateUser,
      updateBackground,
      updateWeather,
      weather,
      backgroundImage,
      backgroundSize,
      backgroundPosition,
      route,
    };
  },
  computed: {
    isAdminPage() {
      return this.$route.path.startsWith("/admin");
    },
  },
});
</script>

<style>
#app {
  min-height: 100vh;
  background-repeat: no-repeat;
  background-color: #f0f0f0; /* Resim yüklenemezse varsayılan arka plan rengi */
}

/* Navbar */
.navbar {
  display: flex;
  align-items: center;
  padding: 1rem 2rem;
  background-color: rgba(0, 0, 0, 0.5);
}

/* Navbar içindeki öğeleri yan yana hizalama */
.nav-links {
  display: flex;
  align-items: center;
  gap: 2rem;
  list-style: none;
  padding: 0;
  margin: 0;
  margin-left: auto; /* Bağlantıları her zaman sağa hizala */
}

/* Bağlantılar ve butonlar */
.navbar a,
.navbar button {
  color: white;
  text-decoration: none;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.25rem;
  padding: 0.5rem 1rem;
  display: flex;
  align-items: center;
}

/* Hover efekti */
.navbar a:hover,
.navbar button:hover {
  text-decoration: underline;
}

/* Kullanıcı bilgileri */
.user-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  color: white;
  font-size: 1.25rem;
}

/* Kullanıcı simgesi */
.user-icon {
  font-size: 1.5rem;
}

/* Hava durumu bilgisi */
.weather-info {
  color: white;
  font-size: 1.25rem;
  background: rgba(0, 0, 0, 0.2);
  padding: 0.5rem 1rem;
  border-radius: 5px;
  display: flex;
  align-items: center;
}

/* Hava durumu ikonu */
.weather-icon {
  width: 40px;
  height: 40px;
  margin-right: 0.1rem;
  margin-top: 0rem;
}
</style>