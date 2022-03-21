<template>
	<div class="weatherapp">
		<div class="weatherapp__input-controls">
			<input
				@keyup.enter="getWeatherOnEnter"
				class="weatherapp__input"
				type="text"
				placeholder="Type a city here."
				v-model="citySearch"
			/>
			<button class="weatherapp__input-button" @click="getWeatherData()">
				<img src="/images/ui-icons/search-icon.svg" alt="" />
			</button>
		</div>

		<div class="weatherapp__output">
			<div class="weatherapp__icon">
				<img
					class="weatherapp__icon--image"
					:src="outputData.iconURL"
					alt="icon of clouds"
				/>
			</div>

			<p class="weatherapp__weather-description">
				{{ outputData.weatherDescription }}
			</p>

			<p class="weatherapp__temperature">{{ outputData.temperature }}</p>

			<p class="weatherapp__city">{{ outputData.cityName }}</p>

			<div class="weatherapp__suggestion-container">
				<h3 class="weatherapp__suggestion-heading">{{ suggestionTitle }}</h3>
				<p class="weatherapp__suggestion" v-html="suggestionHTML"></p>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	data() {
		return {
			htmlString: '<p> hello <span> there </span> </p>',
			citySearch: 'Oslo',
			suggestionTitle: 'Equipment suggestion:',
			suggestionHTML: '',
			outputData: {
				temperature: '',
				weatherDescription: '',
				cityName: '',
				iconURL: '',
				weatherSuggestionsHTML: [
					`<p>Better take on a <span class="weatherapp__suggestion--highlight">winter jacket</span> if you\’re going outside, don\’t want you to freeze!</p>`,

					`<p>
					Better take on a
					<span class="weatherapp__suggestion--highlight">
					light</span>
					to
					<span class="weatherapp__suggestion--highlight">
					medium coat</span>
					 if you’re going outside, don’t want you to freeze!
					</p>`,

					'Should probably wear a <span class="weatherapp__suggestion--highlight">fleece</span> or <span class="weatherapp__suggestion--highlight">warm sweater</span>  before going outside!',

					'It’s going to get warm today, so you’ll be fine with <span class="weatherapp__suggestion--highlight">short sleeves!<span>',

					'It’s going to be hot today, so you should probably wear <span class="weatherapp__suggestion--highlight">shorts</span> or a <span class="weatherapp__suggestion--highlight">flowy dress</span>.',
				],
			},
		};
	},

	created() {
		this.getWeatherData();
	},

	methods: {
		async getWeatherData() {
			try {
				const url = `https://goweather.herokuapp.com/weather/${this.citySearch}`;
				const response = await fetch(url);
				const { temperature, description } = await response.json();

				if (temperature === '' && description === '') {
					this.outputData.cityName = `${this.citySearch} is not a city`;
					this.outputData.temperature = '';
					this.outputData.iconURL = this.loadIcon('error');
					this.outputData.weatherDescription = '';
					this.suggestionHTML = 'We can\'t give suggestions for a city that doesn\'t exist, sadly.'
					return;
				}

				let alteredTemperature = temperature
					.replace(' °C', '°')
					.replace('+', '');

				this.setSuggestion(temperature);

				this.setIconURL(description);

				this.outputData.temperature = this.stringMutation(alteredTemperature);
				this.outputData.weatherDescription = description;
				this.outputData.cityName = this.capitalizeString(this.citySearch);
				this.citySearch = '';
			} catch (error) {
				console.error(
					'⚠️⚠️⚠️⚠️⚠️⚠️ERROR ERROR ERROR DUDE LOOK HERE ERROR!⚠️⚠️⚠️⚠️⚠️' +
						error
				);
				console.error(error.message);
			}
		},

		setIconURL(description) {
			const string = description.toLowerCase();
			const sunnyIcon = string.includes('sun') || string.includes('clear');
			const rainyIcon = string.includes('rain') || string.includes('drizzle');
			const cloudIcon = string.includes('cloud');
			let iconURL = '';

			//Switch (true) here checks each boolean stored in the variables above. So basically if the case is true === true;
			switch (true) {
				case sunnyIcon:
					iconURL = this.loadIcon('sunny');
					break;
				case rainyIcon:
					iconURL = this.loadIcon('rainy');
					break;
				case cloudIcon:
					iconURL = this.loadIcon('cloudy');
					break;
				default:
					iconURL = this.loadIcon('snowing');
			}
			this.outputData.iconURL = iconURL;
		},

		loadIcon(name) {
			return `../../images/weather-app-icons/${name}.svg`;
		},

		//Had to make a separate method for the enter click, because I couldn't get event directly on getWeatherData for some reason, and to use blur() you need event.
		getWeatherOnEnter(event) {
			this.getWeatherData();
			event.target.blur();
		},

		setSuggestion(temperature) {
			//Gets the number from the temperature string.
			let suggestionValue = +temperature.slice(0, temperature.indexOf(' '));

			//Temperature conditionals (An attempt to clean up the if else chain?)
			const fourMinus = suggestionValue <= -4;
			const betweenFourMinusAndSeven =
				suggestionValue > -4 && suggestionValue <= 7;
			const betweenSevenAndEighteen =
				suggestionValue > 7 && suggestionValue <= 18;
			const betweenEighteenAndSeventySix =
				suggestionValue > 18 && suggestionValue <= 26;
			const seventySixAndAbove =
				suggestionValue > 26 && suggestionValue <= 1000;

			if (fourMinus) {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[0];
			} else if (betweenFourMinusAndSeven) {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[1];
			} else if (betweenSevenAndEighteen) {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[2];
			} else if (betweenEighteenAndSeventySix) {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[3];
			} else if (seventySixAndAbove) {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[4];
			} else {
				this.suggestionHTML = this.outputData.weatherSuggestionsHTML[4];
			}
		},

		stringMutation(string) {
			if (string.length < 3) {
				return '0' + string;
			}

			return string;
		},

		capitalizeString(string) {
			return (
				string.toLowerCase().charAt(0).toUpperCase() +
				string.toLowerCase().slice(1)
			);
		},
	},
};
</script>

<style>
.weatherapp {
	font-weight: bold;
}

.weatherapp__input-controls {
	position: relative;
	font-size: 24px;
}

.weatherapp__input {
	background-color: #d2d2d2;
	color: #1e1e1e;
	width: 100%;
	padding: 9px 8px 8px 0px;
	padding-left: calc(
		46px + 8px
	); /* first value here is button width, second is the spacing u want between text and button */
	box-shadow: 0px 3px 4px rgba(0, 0, 0, 0.15);
	/* outline: #ffffff; */
	outline: none;
}

.weatherapp__input:focus {
	background-color: #eeebeb;
}

.weatherapp__input::placeholder {
	opacity: 0.4;
}

.weatherapp__input-button {
	position: absolute;
	top: 0;
	left: 0;
	background: #1e1e1e;
	padding: 0.51em;
}

.weatherapp__input-button:hover {
	background-color: #252525;
}

.weatherapp__input-button img {
	width: 22px;
}

.weatherapp__output {
	display: flex;
	flex-direction: column;
	align-items: center;
	margin-top: 109px;
}

.weatherapp__icon--image {
	width: 48px;
}

.weatherapp__weather-description {
	margin-top: 4px;
	font-size: 14px;
}

.weatherapp__city {
	font-size: 18px;
}

.weatherapp__temperature {
	font-size: 72px;
}

.weatherapp__suggestion-heading {
	width: 100%;
	padding: 8px 32px 8px 32px;
	font-size: 14px;
	margin-top: 50px;
}
.weatherapp__suggestion {
	padding: 8px 32px 8px 32px;
	background-color: #4e4e4e;
	font-weight: normal;
}

.weatherapp__suggestion--highlight {
	font-weight: bold;
	text-decoration: underline;
}

@media screen and (min-width: 800px) {
	.weatherapp__input-controls {
		position: relative;
		font-size: 24px;
		display: block;
		max-width: 819px;
		margin: 0 auto;
	}

	.weatherapp__input {
		border-radius: 2px;
	}

	.weatherapp__icon--image {
		width: 80px;
	}

	.weatherapp__output {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 160px;
	}

	.weatherapp__weather-description {
		font-size: 20px;
		margin-top: 16px;
	}

	.weatherapp__temperature {
		font-size: 100px;
	}

	.weatherapp__city {
		font-size: 24px;
	}

	.weatherapp__suggestion-heading {
		padding-left: 0;
		margin-top: 100px;
		font-size: 20px;
	}
	.weatherapp__suggestion {
		font-size: 24px;
		max-width: 819px;
		padding: 16px 119px;
		border-radius: 2px;
	}
}
</style>
