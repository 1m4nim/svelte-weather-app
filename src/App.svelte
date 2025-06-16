<script>
  let city = "";
  let weather = null;
  let error = "";

  const API_KEY = import.meta.env.VITE_API_KEY;

  async function fetchWeather() {
    if (!city) return;

    const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric&lang=ja`;

    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error("都市が見つかりません");
      weather = await res.json();
      error = "";
    } catch (e) {
      weather = null;
      error = e.message;
    }
  }
</script>

<style>
  .weather-card {
    border: 1px solid #ccc;
    padding: 1em;
    border-radius: 8px;
    max-width: 400px;
    margin: 1em auto;
    text-align: center;
  }
</style>

<h1>天気アプリ</h1>

<input type="text" bind:value={city} placeholder="都市名を入力"  on:keypress={(e) => e.key === 'Enter' && fetchWeather()}/>
<button on:click={fetchWeather}>検索</button>

{#if error}
  <p style="color: red;">{error}</p>
{/if}

{#if weather}
  <div class="weather-card">
    <h2>{weather.name} の天気</h2>
    <p>{weather.weather[0].description}</p>
    <p>気温: {weather.main.temp} ℃</p>
    <p>湿度: {weather.main.humidity} %</p>
    <p>風速: {weather.wind.speed} m/s</p>
  </div>
{/if}
