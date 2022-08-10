<script>
    import { Chart, registerables } from 'chart.js';
    import { onMount } from 'svelte';

    let chartElement;
    const data = [[], [], [], [], [], []];

    let lastUpdate, airTemperature, airHumidity, soilHumidity, particles, wifiSignal;

    async function loadData() {
        const rows = await fetch('http://10.1.2.39:8080/data.csv?t=' + new Date().getTime())
            .then(res => res.text())
            .then(res => res.trim().split('\n').map(row => row.trim().split(' ').map(parseFloat)))
            .then(res => res.slice(-16));

        for (const row of rows) {
            row[0] *= 1000;

            for (let i = 0; i < data.length; ++i) {
                data[i].push(row[i]);
            }
        }

        lastUpdate = new Date(data[0][data[0].length - 1]).toLocaleTimeString('hr');
        airTemperature = `${data[1][data[1].length - 1]}°C`;
        airHumidity = `${data[2][data[2].length - 1]}%`;
        soilHumidity = `${data[3][data[3].length - 1]}%`;
        particles = `${data[4][data[4].length - 1]} µg/m³`
        wifiSignal = `${data[5][data[5].length - 1]}%`;

        const config = { 
            type: 'line',
            animation: false,
            parsing: false,
            data: {
                labels: data[0],
                datasets: [
                    {
                        label: 'air temperature',
                        data: data[1],
                        backgroundColor: '#fa5137',
                        borderColor: '#fa5137',
                        borderWidth: 2,
                    },
                    {
                        label: 'air humidity',
                        data: data[2],
                        backgroundColor: '#37f0fa',
                        borderColor: '#37f0fa',
                        borderWidth: 2,
                    },
                    {
                        label: 'soil humidity',
                        data: data[3],
                        backgroundColor: '#c28025',
                        borderColor: '#c28025',
                        borderWidth: 2,
                    },
                    {
                        label: 'particles',
                        data: data[4],
                        backgroundColor: '#cccccc',
                        borderColor: '#cccccc',
                        borderWidth: 2,
                    },
                    {
                        label: 'WiFi signal',
                        data: data[5],
                        backgroundColor: '#ff00ff',
                        borderColor: '#ff00ff',
                        borderWidth: 2,
                    },
                ]
            },
            options: {
                scales: {
                    x: {
                        ticks: {
                            callback: function(value) {
                                return new Date(this.getLabelForValue(value)).toLocaleTimeString('hr');
                            },
                        }
                    },
                    y: {
                        type: 'linear',
                        min: 0,
                        max: 100
                    }
                }
            } 
        };

        Chart.register(...registerables);
        new Chart(chartElement, config);
    }

    onMount(loadData);
</script>

<div class="container my-5">
    <h1 class="px-4 text-center">AIAgro podaci</h1>
    <div class="row mt-5 mx-0 text-center">
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid red !important">
                {#if lastUpdate}
                    <span>Last update</span>
                    <h2 class="m-0">{lastUpdate}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: red" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}     
            </div>
        </div>
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #fa5137 !important">
                {#if airTemperature}
                    <span>Air temperature</span>
                    <h2 class="m-0">{airTemperature}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #fa5137" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #37f0fa !important">
                {#if airHumidity}
                    <span>Air humidity</span>
                    <h2 class="m-0">{airHumidity}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #37f0fa" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #c28025 !important">
                {#if soilHumidity}
                    <span>Soil humidity</span>
                    <h2 class="m-0">{soilHumidity}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #c28025" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #cccccc !important">
                {#if particles}
                    <span>Particles PM10</span>
                    <h2 class="m-0">{particles}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #cccccc" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #ff00ff !important">
                {#if wifiSignal}
                    <span>WiFi signal</span>
                    <h2 class="m-0">{wifiSignal}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #ff00ff" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    </div>
    <div class="row px-4 py-2">
        <div class="card p-3">
            Ova stranica sadrži sve relevantne podatke senzora (DHT11, HM3301, VMA303) koji se šalju preko MQTT protokola.
            Trenutno ih senzori šalju svakih 5 sekundi, te su odmah dostupni pri osvježavanju stranice.
        </div>
    </div>
    <!--<div class="btn-group px-4" role="group">
        <button type="button" class="btn" style="background-color: #e39e27">1 month</button>
        <button type="button" class="btn" style="background-color: #e39e27">1 day</button>
        <button type="button" class="btn" style="background-color: #e39e27">1 hour</button>
    </div>-->
    <p></p>
    <div class="row px-4">
        <div class="card">
            <canvas bind:this={chartElement} style="height: 50vh; width: 100%"></canvas>
        </div>
    </div>
</div>
