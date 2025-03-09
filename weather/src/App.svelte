<script>
  import { onMount } from "svelte";
  import weatherLogo from "./assets/weather.svg";



  function getLocation() {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition, showError);
    } else {
        console.log("Geolocation is not supported by this browser.");
    }
}

function showPosition(position) {
    const lat = position.coords.latitude;
    const lon = position.coords.longitude;
    getCityFromCoordinates(lat, lon);
}

function showError(error) {
    switch (error.code) {
        case error.PERMISSION_DENIED:
            console.log("User denied the request for Geolocation.");
            break;
        case error.POSITION_UNAVAILABLE:
            console.log("Location information is unavailable.");
            break;
        case error.TIMEOUT:
            console.log("The request to get user location timed out.");
            break;
        case error.UNKNOWN_ERROR:
            console.log("An unknown error occurred.");
            break;
    }
}

function getCityFromCoordinates(lat, lon) {
    const url = `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}&zoom=10&addressdetails=1&accept-language=en`;

    fetch(url)
        .then(response => response.json())
        .then(data => {
            if (data.address && data.address.city) {
                const city = data.address.city;
                console.log(data);
                console.log("User's city:", city);
            } else {
                console.log("City not found.");
            }
        })
        .catch(error => {
            console.log("Error fetching city:", error);
        });
}

getLocation();

  // Note: In a real app, store this in an environment variable, not hardcoded
  const api_key = "3009b74b08fc4276be193415250903";
  let city = "Riyadh"; // Default city
  let forecastData = []; // Reactive variable for forecast data
  let errorMessage = ""; // Reactive variable for errors

  async function getWeather() {
    errorMessage = ""; // Clear previous errors
    const url = `http://api.weatherapi.com/v1/forecast.json?key=${api_key}&q=${city}&days=5`;
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch weather data");
      }
      const data = await response.json();
      forecastData = data.forecast.forecastday; // Update reactive variable
    } catch (error) {
      errorMessage = error.message; // Display error in UI
    }
  }

  // Fetch data when component mounts
  onMount(getWeather);
</script>

<main>
  <div class="header">
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={weatherLogo} class="logo" alt="Weather Logo" />
    </a>
  </div>
  <div>
    <h1>WEATHER</h1>
  </div>

  <!-- Input and button to fetch weather for a custom city -->
  <div>
    <input bind:value={city} placeholder="Enter city" />
    <button on:click={getWeather}>Get Weather</button>
  </div>

  <!-- Display error or table based on state -->
  {#if errorMessage}
    <p>{errorMessage}</p>
  {:else}
    <div class="table-container">
      <table>
        <caption>5-Day Weather Forecast for {city}</caption>
        <thead>
          <tr>
            <th scope="col">Date</th>
            <th scope="col">Temperature</th>
            <th scope="col">Humidity</th>
            <th scope="col">Wind Speed</th>
            <th scope="col">Condition</th>
          </tr>
        </thead>
        <tbody>
          {#each forecastData as day}
            <tr>
              <td>{day.date}</td>
              <td>{day.day.avgtemp_c}°C -  {day.day.avgtemp_f}°F</td>
              <td>{day.day.avghumidity}%</td>
              <td>{day.day.maxwind_kph} km/h</td>
              <td>
                {day.day.condition.text}
                <div>
                  <img
                    src={`https:${day.day.condition.icon}`}
                    alt="weather icon"
                  />
                </div>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/if}
</main>

<style>
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .logo {
    height: 6em;
    transition: filter 300ms;
  }

  .logo:hover {
    filter: drop-shadow(0 0 2em #3498db);
  }

  h1 {
    font-size: 2.5rem;
    color: #2c3e50;
  }

  .table-container {
    width: 100%;
    max-width: 800px;
    overflow-x: auto;
  }

  table {
    border-collapse: collapse;
    width: 100%;
    font-family: sans-serif;
  }

  th,
  td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
  }

  th {
    background-color: #3498db;
    color: white;
    font-weight: bold;
  }

  tr:nth-child(even) {
    background-color: #f9f9f9;
  }

  tr:hover {
    background-color: #ecf0f1;
  }

  caption {
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
    color: #2c3e50;
  }

  @media (max-width: 600px) {
    table {
      font-size: 0.8rem;
    }
    th,
    td {
      padding: 8px;
    }
  }
</style>
