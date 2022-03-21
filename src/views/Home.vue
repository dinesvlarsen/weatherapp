<template>
	<WeatherApp />
</template>

<script>
import WeatherApp from '../components/WeatherApp.vue';
export default {
	data() {
		return {
			appName: import.meta.env.VITE_APP_NAME,
		};
	},

	components: {
		WeatherApp,
	},
};
</script>
