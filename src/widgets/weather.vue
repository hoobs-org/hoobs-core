<template>
    <div v-if="weather && forecast" id="weather">
        <div class="current-weather">
            <span class="current-title">{{ $t(forecastDay(new Date())) }}</span>
            <span class="current-time">{{ formatTime(new Date()) }}</span>
            <div class="current-weather-icon" :class="`wi wi-day-${icon[weather.data.weather[0].id].icon}`"></div>
            <span class="current-description">{{ $t(icon[weather.data.weather[0].id].label) }}</span>
            <span class="current-temp">{{ Math.round(weather.data.main.temp) }}°</span>
        </div>
        <div class="details">
            <div class="detail-contents">
                <span class="detail-title">{{ $t("forecast") }}</span>
            </div>
            <div class="forecast">
                <div v-for="(day, index) in forecast.data.list" :key="index">
                    <div v-if="isNoon(day.dt_txt)" class="forecast-weather">
                        <span class="forecast-title">{{ $t(forecastDay(day.dt_txt)) }}</span>
                        <div class="forecast-weather-icon" :class="`wi wi-day-${icon[day.weather[0].id].icon}`"></div>
                        <span class="forecast-description">{{ $t(icon[day.weather[0].id].label) }}</span>
                        <span class="forecast-temp">{{ Math.round(day.main.temp) }}°</span>
                    </div>
                </div>
            </div>
        </div>
        <div class="actions">
            <span class="action-label">°C</span>
            <span class="icon" v-on:click="toggleUnits()">{{ toggleIcon }}</span>
            <span class="action-label">°F</span>
        </div>
    </div>
</template>

<script>
    import "../assets/weather-icons.css";

    import Request from "jsonp";
    import Dates from "../dates";
    import WeatherIcons from "../weather-icons.json";

    export default {
        name: "weather",

        props: {
            item: Object,
            index: Number,
            change: Function
        },

        data() {
            return {
                units: "imperial",
                toggleIcon: "toggle_on"
            }
        },

        computed: {
            weather() {
                return this.$store.state.weather;
            },

            forecast() {
                return this.$store.state.forecast;
            },

            icon() {
                return WeatherIcons;
            }
        },

        async mounted() {
            this.units = this.item.units === "metric" ? "metric" : "imperial";
            this.toggleIcon = this.units === "metric" ? "toggle_off" : "toggle_on";

            if (!this.weather || new Date().getTime() - this.weather.date.getTime() >= 3600000) {
                this.loadWeather(await this.getQuery());
            }

            if (!this.forecast || new Date().getTime() - this.weather.date.getTime() >= 86400000) {
                this.loadForecast(await this.getQuery());
            }
        },

        methods: {
            loadWeather(query) {
                console.log(query);
                Request(`https://api.openweathermap.org/data/2.5/weather?${query}&units=${this.units}&appid=${atob("ZmVjNjdiNTVmN2Y3NGRlYWEyOGRmODliYTZhNjA4MjE=")}`, null, (error, response) => {
                    if (!error) {
                        this.$store.commit("current", response);
                    }
                });
            },

            loadForecast(query) {
                Request(`https://api.openweathermap.org/data/2.5/forecast?${query}&units=${this.units}&appid=${atob("ZmVjNjdiNTVmN2Y3NGRlYWEyOGRmODliYTZhNjA4MjE=")}`, null, (error, response) => {
                    if (!error) {
                        this.$store.commit("future", response);
                    }
                });
            },

            async toggleUnits() {
                if (this.item.units === "metric") {
                    this.units = "imperial";
                    this.toggleIcon = "toggle_on";
                } else {
                    this.units = "metric";
                    this.toggleIcon = "toggle_off";
                }

                this.item.units = this.units;

                this.change(this.index, "units", this.units);

                this.loadWeather(await this.getQuery());
                this.loadForecast(await this.getQuery());
            },

            forecastDay(value) {
                return Dates.getWeekDayName(value);
            },

            formatTime(value) {
                return Dates.formatTime(value);
            },

            isNoon(value) {
                const date = new Date(value);

                if (date.getHours() === 12 && date.getMinutes() === 0) {
                    return true;
                }

                return false;
            },

            getQuery() {
                return new Promise((resolve) => {
                    const query = this.$cookie("weather_query");

                    if (query && query !== "undefined" && query !== "") {
                        resolve(query.replace(/\|/gi, "="));
                    } else {
                        let results = null;

                        this.geolocation().then((position) => {
                            Request(`https://api.openweathermap.org/data/2.5/find?lat=${position.latitude}&lon=${position.longitude}&cnt=10&appid=${atob("ZmVjNjdiNTVmN2Y3NGRlYWEyOGRmODliYTZhNjA4MjE=")}`, null, (error, response) => {
                                if (!error && response.list && Array.isArray(response.list) && response.list.length > 0) {
                                    results = `id|${response.list[0].id}`;
                                } else {
                                    results = `lat|${position.latitude}&lon=${position.longitude}`;
                                }

                                this.$cookie("weather_query", results, 20160);

                                resolve(results.replace(/\|/gi, "="));
                            });
                        });
                    }
                });
            },

            geolocation() {
                return new Promise((resolve) => {
                    const latitude = parseFloat(this.$cookie("latitude"));
                    const longitude = parseFloat(this.$cookie("longitude"));

                    if (!Number.isNaN(latitude) && !Number.isNaN(longitude)) {
                        resolve({
                            latitude,
                            longitude
                        });
                    } else {
                        const position = {
                            latitude: 40.59,
                            longitude: -105.08
                        };

                        Request("http://ip-api.com/json/", null, (error, response) => {
                            if (!error) {
                                this.$cookie("latitude", response.lat, 20160);
                                this.$cookie("longitude", response.lon, 20160);

                                position.latitude = response.lat;
                                position.longitude = response.lon;
                            }

                            resolve(position);
                        });
                    }
                });
            }
        }
    };
</script>

<style scoped>
    #weather {
        height: 100%;
        padding: 0 20px;
        display: flex;
        align-content: center;
        align-items: center;
        justify-content: space-between;
        position: relative;
    }

    #weather .current-weather {
        width: 200px;
        text-align: center;
        display: flex;
        flex-direction: column;
        align-items: center;
        align-content: center;
        box-sizing: border-box;
    }

    #weather .current-weather-icon {
        font-size: 60px;
        line-height: 100px;
        color: var(--title-text);
    }

    #weather .current-title {
        font-weight: bold;
        font-size: 20px;
        color: var(--title-text);
    }
    
    #weather .current-description {
        font-weight: bold;
        font-size: 20px;
    }

    #weather .current-temp {
        font-weight: bold;
        font-size: 28px;
    }

    #weather .current-time {
        margin: 0 0 10px 0;
    }

    #weather .details {
        flex: 1;
        display: flex;
        flex-direction: column;
    }

    #weather .details .detail-contents {
        display: flex;
        border-bottom: 1px var(--border) solid;
        justify-content: space-around;
        margin: 0 0 20px 0;
        padding: 0 20px 10px 20px;
    }

    #weather .detail-title {
        font-weight: bold;
        font-size: 18px;
        color: var(--title-text);
    }

    #weather .forecast {
        flex: 1;
        display: flex;
        justify-content: space-between;
        padding: 0 20px 0 0;
        overflow: auto;
    }

    #weather .forecast-weather {
        width: 100px;
        text-align: center;
        display: flex;
        flex-direction: column;
        align-items: center;
        align-content: center;
        box-sizing: border-box;
    }

    #weather .forecast-weather-icon {
        font-size: 40px;
        line-height: 80px;
        color: var(--title-text);
    }

    #weather .forecast-title {
        font-weight: bold;
        font-size: 14px;
    }
    
    #weather .forecast-description {
        font-weight: bold;
        font-size: 12px;
    }

    #weather .forecast-temp {
        font-weight: bold;
        font-size: 18px;
    }

    #weather .actions {
        font-size: 14px;
        position: absolute;
        top: 20px;
        right: 20px;
        display: flex;
        align-content: center;
        align-items: center;
        user-select: none;
    }

    #weather .action-label {
        font-size: 12px;
        margin: 0 5px;
    }

    #weather .actions .icon {
        font-size: 28px;
        color: var(--title-text);
        cursor: pointer;
    }
</style>