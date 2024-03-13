<script setup>

import { ref, onMounted } from 'vue';
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);


const props = defineProps({
   dados: Object
});

const chartRef = ref(null);
const dados = props.dados;


function montarGrafico(dados) {
    if (chartRef.value && dados) {
        const ctx = chartRef.value.getContext('2d');
        new Chart(ctx, {
            type: 'line',
            data: {
                labels: dados.labels,
                datasets: [
                    {
                        label: 'Precipitação acumulada (mm)',
                        data: dados.datasets[0].data,
                        backgroundColor: dados.datasets[0].backgroundColor,
                        borderColor: dados.datasets[0].borderColor,
                        borderWidth: dados.datasets[0].borderWidth
                    },
                    {
                        label: 'Temperatura Média',
                        data: dados.datasets[1].data,
                        type: 'line',
                        backgroundColor: dados.datasets[1].backgroundColor,
                        borderColor: dados.datasets[1].borderColor,
                        borderWidth: dados.datasets[1].borderWidth,
                        yAxisID: 'y-axis-1'
                    }
                ]
            },
            options: {
                scales: {
                    y: {
                        type: 'linear',
                        display: true,
                        position: 'left',
                        title: {
                            display: true,
                            text: 'Precipitação (mm)'
                        }
                    },
                    y1: {
                        type: 'linear',
                        display: true,
                        position: 'right',
                        title: {
                            display: true,
                            text: 'Temperatura (°C)'
                        }
                    }
                }
            }
        });
    }
}

onMounted(() => {
    montarGrafico(dados);
});
</script>

<template>
    <div>
        {{ dados }}
        <canvas ref="chartRef" width="400" height="400"></canvas>
    </div>
</template>