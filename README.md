# Urjashakti<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Urjashakti | Live PV Forecast</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; }
        .main-gradient-text { background: linear-gradient(to right, #facc15, #fb923c); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .card { background-color: #1e293b; border: 1px solid #334155; transition: transform 0.3s ease, box-shadow 0.3s ease; animation: fadeIn 0.5s ease-out forwards; }
        .card:hover { transform: translateY(-5px); box-shadow: 0 10px 20px rgba(0,0,0,0.2); }
        .recommendation-card { background-color: #1e293b80; border: 1px solid #334155; }
        .icon-bg { background-color: #334155; }
        .chart-container { position: relative; width: 100%; height: 24rem; max-height: 400px; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
        .live-dot { animation: pulse 2s infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }
    </style>
</head>
<body class="text-gray-200">
    <div class="container mx-auto p-4 md:p-8">
        <header class="text-center mb-10">
            <h1 class="text-4xl md:text-5xl font-extrabold main-gradient-text">Urjashakti</h1>
            <p id="greeting" class="text-xl text-slate-300 mt-4">Intelligent Solar Power Forecast</p>
             <div class="flex items-center justify-center flex-wrap gap-4 mt-4 text-slate-400">
                <div class="flex items-center gap-2"><div class="w-3 h-3 bg-green-500 rounded-full live-dot"></div>Live</div>
                <span id="current-time"></span><span>-</span><span id="current-date"></span><span>-</span>
                <div class="flex items-center gap-1">
                    <span class="text-yellow-400">&#9728;</span>
                    <span>San Diego, CA</span>
                </div>
            </div>
        </header>
        <main class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <div class="lg:col-span-2 card rounded-2xl p-6">
                <h2 class="text-2xl font-bold text-white mb-1">24-Hour Generation Forecast</h2>
                <p class="text-slate-400 mb-6">This chart shows the predicted solar power output in kilowatts (kW) for the next 24 hours. Hover over any point on the line to see the specific forecast for that hour.</p>
                <div class="chart-container">
                    <canvas id="forecastChart"></canvas>
                </div>
            </div>
            <div class="flex flex-col gap-8">
                <div class="card rounded-2xl p-6 flex flex-col items-center text-center" style="animation-delay: 100ms;">
                    <h3 class="text-lg font-semibold text-slate-400">Peak Generation</h3>
                    <p class="text-5xl font-bold main-gradient-text my-2" id="peak-power">-- kW</p>
                    <p class="text-lg text-white" id="peak-time">at --:--</p>
                </div>
                <div class="card rounded-2xl p-6 flex flex-col items-center text-center" style="animation-delay: 200ms;">
                     <h3 class="text-lg font-semibold text-slate-400">Optimal Usage Window</h3>
                     <p class="text-3xl md:text-4xl font-bold text-white my-2" id="optimal-window">--:-- - --:--</p>
                     <p class="text-slate-300">Best time for high-power tasks</p>
                </div>
                 <div class="card rounded-2xl p-6 flex flex-col items-center text-center" style="animation-delay: 300ms;">
                    <h3 class="text-lg font-semibold text-slate-400">Total Estimated Generation</h3>
                    <p class="text-5xl font-bold main-gradient-text my-2" id="total-kwh">--</p>
                    <p class="text-lg text-white">kWh / day</p>
                </div>
            </div>
        </main>
        <section class="mt-12">
            <h2 class="text-3xl font-bold text-center mb-8 text-white">Automated Recommendations</h2>
            <p class="text-center text-slate-400 max-w-3xl mx-auto mb-8">Based on the forecast, here are intelligent suggestions to help you maximize solar energy self-consumption and reduce your reliance on the grid. Recommendations are activated based on the predicted peak power.</p>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="recommendations-grid"></div>
        </section>
    </div>
    <script>
        document.addEventListener('DOMContentLoaded', function () {
            const timeEl = document.getElementById('current-time');
            const dateEl = document.getElementById('current-date');
            const greetingEl = document.getElementById('greeting');
            function updateTime() {
                const now = new Date();
                const hours = now.getHours();
                if (hours < 12) { greetingEl.textContent = 'Good Morning! Here is your solar forecast.'; }
                else if (hours < 18) { greetingEl.textContent = 'Good Afternoon! Here is your solar forecast.'; }
                else { greetingEl.textContent = 'Good Evening! Here is your solar forecast.'; }
                timeEl.textContent = now.toLocaleTimeString('en-US', { timeZone: 'America/Los_Angeles', hour: '2-digit', minute: '2-digit' });
                dateEl.textContent = now.toLocaleDateString('en-US', { timeZone: 'America/Los_Angeles', weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
            }
            updateTime();
            setInterval(updateTime, 1000);

            const forecastData = {
                hours: Array.from({ length: 24 }, (_, i) => `${String(i).padStart(2, '0')}:00`),
                predicted_kw: [0.68, 1.27, 0.78, 0.0, 0.0, 0.0, 0.0, 4.03, 14.06, 28.0, 40.0, 48.5, 51.05, 49.33, 40.38, 27.78, 14.26, 4.11, 0.0, 0.0, 0.0, 0.0, 0.0, 0.61]
            };

            const peakPower = Math.max(...forecastData.predicted_kw);
            const peakHourIndex = forecastData.predicted_kw.indexOf(peakPower);
            const peakTime = forecastData.hours[peakHourIndex];
            const totalKwh = forecastData.predicted_kw.reduce((sum, current) => sum + (current > 0 ? current : 0), 0);
            const highPowerThreshold = peakPower * 0.70;
            const optimalHours = forecastData.hours.filter((hour, index) => forecastData.predicted_kw[index] >= highPowerThreshold);
            const optimalWindow = optimalHours.length > 0 && peakPower > 0 ? `${optimalHours[0]} - ${String(parseInt(optimalHours[optimalHours.length - 1].split(':')[0]) + 1).padStart(2, '0')}:00` : "N/A";

            document.getElementById('peak-power').textContent = `${peakPower.toFixed(2)} kW`;
            document.getElementById('peak-time').textContent = `at ${peakTime}`;
            document.getElementById('optimal-window').textContent = optimalWindow;
            document.getElementById('total-kwh').textContent = totalKwh.toFixed(1);

            const recommendationsGrid = document.getElementById('recommendations-grid');
            const recommendations = [
                { icon: '⚡️', title: 'EV Charging', description: `Schedule charging between ${optimalWindow} to use 100% clean energy and reduce costs.`, active: peakPower > 3 },
                { icon: '💨', title: 'Air Conditioning', description: `Pre-cool your space during the optimal window (${optimalWindow}) to minimize grid usage later.`, active: peakPower > 2 },
                { icon: '💧', title: 'Water Pump / Pool Filter', description: `Run pumps and filters between ${optimalWindow} when energy is most abundant.`, active: peakPower > 1.5 },
                { icon: '🧺', title: 'Home Appliances', description: 'Use the dishwasher, washing machine, or dryer during high-generation periods to maximize solar self-consumption.', active: true },
                { icon: '🔋', title: 'Battery Storage', description: 'Prioritize charging your home battery system during peak hours for use overnight.', active: peakPower > 1 },
                { icon: '📉', title: 'Low Generation Alert', description: 'Solar output is low. Shift non-essential power usage to peak hours tomorrow to save on grid electricity costs.', active: peakPower < 1 }
            ];
            
            recommendationsGrid.innerHTML = '';
            const activeRecommendations = recommendations.filter(rec => rec.active);
            if (activeRecommendations.length === 0) {
                recommendationsGrid.innerHTML = `<p class="text-slate-400 text-center col-span-full">No specific recommendations at this moment.</p>`;
            } else {
                activeRecommendations.forEach((rec, index) => {
                    const cardHTML = `
                        <div class="recommendation-card rounded-2xl p-6 flex items-start gap-4 card" style="animation-delay: ${index * 100 + 400}ms;">
                            <div class="icon-bg p-3 rounded-full text-2xl">${rec.icon}</div>
                            <div><h3 class="text-xl font-bold text-white">${rec.title}</h3><p class="text-slate-400">${rec.description}</p></div>
                        </div>`;
                    recommendationsGrid.innerHTML += cardHTML;
                });
            }
            
            const ctx = document.getElementById('forecastChart').getContext('2d');
            const gradient = ctx.createLinearGradient(0, 0, 0, 400);
            gradient.addColorStop(0, 'rgba(250, 204, 21, 0.6)');
            gradient.addColorStop(1, 'rgba(15, 23, 42, 0.1)');

            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: forecastData.hours,
                    datasets: [{
                        label: 'Predicted Power (kW)',
                        data: forecastData.predicted_kw,
                        borderColor: '#facc15', borderWidth: 3, pointBackgroundColor: '#facc15',
                        pointHoverRadius: 7, pointHoverBorderColor: '#fff', tension: 0.4,
                        fill: true, backgroundColor: gradient,
                    }]
                },
                options: {
                    responsive: true, maintainAspectRatio: false,
                    scales: {
                        y: { beginAtZero: true, grid: { color: '#334155' }, ticks: { color: '#94a3b8', font: { size: 14 } }, title: { display: true, text: 'Power (kW)', color: '#cbd5e1' } },
                        x: { grid: { display: false }, ticks: { color: '#94a3b8', font: { size: 14 }, maxRotation: 0, autoSkip: true, maxTicksLimit: 12 } }
                    },
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            enabled: true, backgroundColor: '#0f172a', titleFont: { size: 16, weight: 'bold' },
                            bodyFont: { size: 14 }, padding: 12, borderColor: '#334155', borderWidth: 1,
                            callbacks: { label: (context) => `Power: ${context.parsed.y.toFixed(2)} kW` }
                        }
                    }
                }
            });
        });
    </script>
</body>
</html># Urjashakti - Intelligent Solar Power Forecast ☀️

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)](https://www.tensorflow.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)

**Urjashakti** is a machine learning-powered web dashboard that provides a 24-hour forecast for solar power generation. Developed for the Smart India Hackathon, this tool uses a CNN-LSTM model to predict PV output based on historical and weather data, offering actionable recommendations for optimal energy consumption.
--
---

### ✨ Key Features

* **24-Hour Forecasting:** Utilizes a robust CNN-LSTM deep learning model to predict solar generation with high accuracy.
* **Interactive Dashboard:** A sleek, user-friendly interface built with HTML and Tailwind CSS to visualize the complex forecast data.
* **Automated Recommendations:** Provides intelligent suggestions for energy usage (e.g., scheduling EV charging, running appliances) based on predicted peak generation times.
* **Live Data Metrics:** Displays key performance indicators like Peak Power (kW), Optimal Usage Window, and Total Daily Generation (kWh).
* **End-to-End Pipeline:** A complete Python script handles data fetching, model training, prediction, and dynamic dashboard generation.

---

### ⚙️ How It Works

The project follows a complete data science workflow from data acquisition to deployment:

1.  **Data Acquisition:** Weather data is fetched from the Meteostat API, and historical PV generation data is loaded.
2.  **Data Preprocessing:** The data is cleaned, merged, and normalized to prepare it for the model.
3.  **Model Training:** A CNN-LSTM model is trained on the time-series data to learn temporal patterns.
4.  **Prediction:** The trained model predicts the next 24 hours of solar power generation.
5.  **Dashboard Generation:** A Python script dynamically injects the forecast data into an HTML template, creating a self-contained, interactive dashboard ready for deployment.

---

### 🛠️ Technology Stack

* **Backend & Model:** Python, TensorFlow, Keras, Scikit-learn, Pandas
* **Frontend:** HTML, Tailwind CSS, Chart.js
* **Deployment:** GitHub Pages
