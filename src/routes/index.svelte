<script>
    import { Chart, registerables } from 'chart.js';
    import { page } from '$app/stores'
    import { onMount } from 'svelte';

    let chartElement, chargeChartElement;
    const data = [[], [], [], [], [], [], []];

    let lastUpdate, airTemperature, airHumidity, soilHumidity, particles, wifiSignal, charge;

    async function loadData() {
        const last = parseInt($page.url.searchParams.get('last')) || 15;
        const dataset = $page.url.searchParams.get('dataset') || 'data';

        const rows = await fetch(`http://10.1.2.40:8080/${dataset}.csv?t=${new Date().getTime()}`)
            .then(res => res.text())
            .then(res => res.trim().split('\n').map(row => row.trim().split(' ').map(parseFloat)))
            .then(res => res.slice(-last));

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
        charge = `${data[6][data[6].length - 1]} V`;

        const config = { 
            type: 'line',
            animation: false,
            parsing: false,
            data: {
                labels: data[0].map(t => new Date(t).toUTCString()),
                datasets: [
                    {
                        label: 'Air temperature',
                        data: data[1],
                        backgroundColor: '#fa5137',
                        borderColor: '#fa5137',
                        borderWidth: 2,
                    },
                    {
                        label: 'Air humidity',
                        data: data[2],
                        backgroundColor: '#37f0fa',
                        borderColor: '#37f0fa',
                        borderWidth: 2,
                    },
                    {
                        label: 'Soil humidity',
                        data: data[3],
                        backgroundColor: '#c28025',
                        borderColor: '#c28025',
                        borderWidth: 2,
                    },
                    {
                        label: 'Particles',
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
                    }
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
                },
                elements: {
                    line: {
                        tension: 0.5
                    }
                }
            } 
        };

        Chart.register(...registerables);

        new Chart(chargeChartElement, {
            type: 'line',
            animation: false,
            parsing: false,
            data: {
                labels: data[0].map(t => new Date(t).toUTCString()),
                datasets: [
                    {
                        label: 'Charge',
                        data: data[6],
                        backgroundColor: '#00ff4080',
                        borderColor: '#00ff40',
                        borderWidth: 2,
                        fill: true,
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
                        max: 5
                    }
                },
                elements: {
                    line: {
                        tension: 0.5
                    }
                }
            }
        });

        new Chart(chartElement, config);
    }

    onMount(loadData);
</script>

<div class="container my-5">
    <h1 class="px-4 mb-4 text-center">AIAgro podaci</h1>
    <div class="row px-4 mb-2">
        <div class="card p-3">
            Ova stranica sadrži sve relevantne podatke senzora (DHT11, HM3301, VMA303) koji se šalju preko MQTT protokola.
            Trenutno ih senzori šalju svakih 5 sekundi, te su odmah dostupni pri osvježavanju stranice.
        </div>
    </div>
    <div class="row mx-0 text-center">
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
        <div class="col-md-4 mb-2">
            <div class="card p-3" style="border-left: 2px solid #00ff40 !important">
                {#if charge}
                    <span>Charge</span>
                    <h2 class="m-0">{charge}</h2>
                {:else}
                    <div class="text-center">
                        <div class="spinner-grow" style="width: 1rem; height: 1rem; color: #00ff40" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    </div>
    <div class="row px-4 py-2 mb-2">
        <div class="card">
            <canvas bind:this={chartElement} style="height: 50vh; width: 100%"></canvas>
        </div>
    </div>
    <div class="row px-4 py-2 mb-2">
        <div class="card">
            <canvas bind:this={chargeChartElement} style="height: 50vh; width: 100%"></canvas>
        </div>
    </div>
</div>
