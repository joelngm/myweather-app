<script>
  const apiKey = "be82b200cefb710fa0c0460b71c1aecb";

  let cityInput = "";
  let loading = false;

  let weather = null;
  let error = "";

  /* ---------------- GEO API ---------------- */
  async function getCoordinates(city) {
    try {
      const res = await fetch(
        `https://api.openweathermap.org/geo/1.0/direct?q=${encodeURIComponent(
          city
        )}&limit=1&appid=${apiKey}`
      );
      const data = await res.json();

      if (!data || data.length === 0) {
        error = "❌ City not found";
        return null;
      }

      return {
        lat: data[0].lat,
        lon: data[0].lon,
        name: data[0].name,
        country: data[0].country
      };
    } catch {
      error = "❌ Failed to fetch coordinates";
      return null;
    }
  }

  /* ---------------- WEATHER API ---------------- */
  async function fetchWeather(lat, lon, locationName) {
    loading = true;
    error = "";
    weather = null;

    try {
      const res = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric`
      );
      const data = await res.json();

      weather = {
        location: locationName,
        temp: data.main.temp,
        description: data.weather[0].description,
        humidity: data.main.humidity,
        wind: data.wind.speed
      };
    } catch {
      error = "❌ Failed to fetch weather data";
    } finally {
      loading = false;
    }
  }

  /* ---------------- CITY SEARCH ---------------- */
  async function handleCitySearch() {
    if (!cityInput.trim()) return;

    const coords = await getCoordinates(cityInput);
    if (coords) {
      fetchWeather(
        coords.lat,
        coords.lon,
        `${coords.name}, ${coords.country}`
      );
    }
  }

  /* ---------------- GEOLOCATION ---------------- */
  function handleLocation() {
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        fetchWeather(
          pos.coords.latitude,
          pos.coords.longitude,
          "My Location"
        );
      },
      () => {
        error = "❌ Location access denied";
      }
    );
  }
</script>

<main class="container">
  <h1>Global Weather App 🌍</h1>

  <input
    placeholder="Enter city name"
    bind:value={cityInput}
    on:keydown={(e) => e.key === "Enter" && handleCitySearch()}
  />

  <button on:click={handleCitySearch}>Get Weather</button>
  <button on:click={handleLocation}>Use My Location</button>

  {#if loading}
    <p class="loading">⏳ Loading...</p>
  {/if}

  {#if error}
    <p class="error">{error}</p>
  {/if}

  {#if weather}
    <div class="weather-result">
      <h2>{weather.location}</h2>
      <p>🌡 Temperature: {weather.temp} °C</p>
      <p>☁ Weather: {weather.description}</p>
      <p>💧 Humidity: {weather.humidity}%</p>
      <p>💨 Wind: {weather.wind} m/s</p>
    </div>
  {/if}
</main>

<style>
  body {
    margin: 0;
  }

  .container {
    min-height: 100vh;
    background: linear-gradient(to right, #74ebd5, #9face6);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-family: Arial, sans-serif;
    padding: 2rem;
  }

  input,
  button {
    width: 100%;
    max-width: 320px;
    padding: 0.6rem;
    margin-bottom: 0.5rem;
    font-size: 1rem;
  }

  button {
    border: none;
    border-radius: 6px;
    background: #4a6cf7;
    color: white;
    cursor: pointer;
  }

  button:hover {
    background: #3955c6;
  }

  .weather-result {
    margin-top: 1rem;
    background: rgba(255, 255, 255, 0.9);
    padding: 1rem;
    border-radius: 10px;
    width: 100%;
    max-width: 320px;
    text-align: center;
  }

  .loading {
    margin-top: 1rem;
  }

  .error {
    margin-top: 1rem;
    color: red;
    font-weight: bold;
  }
</style>
