<script>
    import { Chart, registerables } from 'chart.js';
    import 'chartjs-adapter-moment';
    import { onMount } from 'svelte';

    let chartElement;
    const data = [[], [], [], [], []];

    let lastUpdate, airTemperature, airHumidity, soilHumidity, dust;

    async function loadData() {
        const rows = await fetch('http://10.1.2.39:8080/data.csv?t=' + new Date().getTime())
            .then(res => res.text())
            .then(res => res.trim().split('\n').map(row => row.trim().split(' ').map(parseFloat)))
            .then(res => res.slice(-16));

        for (const row of rows) {
            row[0] *= 1000;

            for (let i = 0; i < 5; ++i) {
                data[i].push(row[i]);
            }
        }

        lastUpdate = new Date(data[0][data[0].length - 1]).toLocaleTimeString('hr');
        airTemperature = `${data[1][data[1].length - 1]}°C`;
        airHumidity = `${data[2][data[2].length - 1]}%`;
        soilHumidity = `${data[3][data[3].length - 1]}%`;
        dust = `${data[4][data[4].length - 1]} µg/m³`

        const config = { 
            type: 'line',
            animation: false,
            parsing: false,
            data: {
                labels: data[0],
                datasets: [
                    {label: 'air temperature', data: data[1], backgroundColor: '#fa5137'},
                    {label: 'air humidity', data: data[2], backgroundColor: '#37f0fa'},
                    {label: 'soil humidity', data: data[3], backgroundColor: '#c28025'},
                    {label: 'dust', data: data[4], backgroundColor: '#cccccc'},
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
        <div class="col-md mb-2">
            <div class="card p-3" style="border-left: 2px solid red !important">
                {#if lastUpdate}
                    {lastUpdate}
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: red" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}     
            </div>
        </div>
        <div class="col-md mb-2">
            <div class="card p-3" style="border-left: 2px solid #fa5137 !important">
                {#if airTemperature}
                    {airTemperature}
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #fa5137" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md mb-2">
            <div class="card p-3" style="border-left: 2px solid #37f0fa !important">
                {#if airHumidity}
                    {airHumidity}
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #37f0fa" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md mb-2">
            <div class="card p-3" style="border-left: 2px solid #c28025 !important">
                {#if soilHumidity}
                    {soilHumidity}
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #c28025" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
        <div class="col-md mb-2">
            <div class="card p-3" style="border-left: 2px solid #cccccc !important">
                {#if soilHumidity}
                    {soilHumidity}
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #cccccc" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    </div>
    <div class="row p-4">
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
    <div class="row p-4">
        <div class="card">
            <canvas bind:this={chartElement} style="height: 50vh; width: 100%"></canvas>
        </div>
    </div>
</div>
