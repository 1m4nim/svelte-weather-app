<script>
  import { onMount } from 'svelte';
  import L from 'leaflet';

  let city = "";
  let weather = null;
  let error = "";
  let map;
  let marker;

  const API_KEY = import.meta.env.VITE_API_KEY;

  onMount(() => {
    // 初期地図（東京）
    map = L.map('map').setView([35.681236, 139.767125], 5);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);
  });

  async function fetchWeather() {
    if (!city) return;

    const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric&lang=ja`;

    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error("都市が見つかりません");

      weather = await res.json();
      error = "";

      const { lat, lon } = weather.coord;

      // 地図を更新
      map.setView([lat, lon], 10);

      // すでにマーカーがある場合は削除
      if (marker) {
        map.removeLayer(marker);
      }

      // マーカー追加
      marker = L.marker([lat, lon]).addTo(map)
        .bindPopup(`
          <b>${weather.name}</b><br>
          天気: ${weather.weather[0].description}<br>
          気温: ${weather.main.temp} ℃<br>
          湿度: ${weather.main.humidity} %<br>
          風速: ${weather.wind.speed} m/s
        `).openPopup();
    } catch (e) {
      weather = null;
      error = e.message;
    }
  }
</script>

<h1>天気 & 地図アプリ</h1>

<input type="text" bind:value={city} placeholder="都市名を入力" />
<button on:click={fetchWeather}>検索</button>

{#if error}
  <p style="color: red;">{error}</p>
{/if}

{#if weather}
  <div>
    <h2>{weather.name} の天気</h2>
    <p>{weather.weather[0].description}</p>
    <p>気温: {weather.main.temp} ℃</p>
    <p>湿度: {weather.main.humidity} %</p>
    <p>風速: {weather.wind.speed} m/s</p>
  </div>
{/if}

<div id="map"></div>
