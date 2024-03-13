<script setup>
import { ref, onMounted } from 'vue';

import 'leaflet';
import 'leaflet/dist/leaflet.css';
import 'proj4';
import 'proj4leaflet';
import 'leaflet-mouse-position';
import 'leaflet.vectorgrid';
import '../../../../css/app.css'

import {
    ConfigsLeaflet,
    AdicionaCoordenadasMouse,
    AdicionaEscala,
    FuncaoVisaoInicial,
    AdicionaAttribution,
    CriaMenuContexto,
    FuncaoMapaInformacoes,
    FuncoesZoom 
} from '@/Pages/Mapa/Functions/InicializarMapa.js';

import { AdicionaBasemapPadrao } from '@/Pages/Mapa/Functions/TileLayers/AdicionaBasemapPadrao.js';
import { AdicionaOverlaysPadrao } from '@/Pages/Mapa/Functions/TileLayers/AdicionaOverlaysPadrao.js';
import { ToggleRasterTile } from '@/Pages/Mapa/Functions/TileLayers/ToggleRasterTile.js';

const props = defineProps({
    projeto: Object,
    rasters: Object
});

let configsMapa = props.projeto.mapa.configuracao;

let rasters = props.projeto.rasters;

let map = ref(null);

// Declarada nova função para conseguir acessar propriedades da escala, como posição
function AdicionarEscala(map, escala) {
    L.control.scale({ imperial: false, metric: true, maxWidth: 200, position: 'bottomright', }).addTo(map);
}

// Declarada nova função para conseguir acessar propriedades da coordenada, como posição e adicionar uma classe
function AdicionarCoordenadasMouse(map, configs) {
    const coordinateLayout = L.control.mousePosition({ position: 'bottomleft', separator: ' ___ ', emptyString: 'Coordenadas: N/D', numDigits: 5 }).addTo(map);
    // Adicionando uma classe personalizada ao elemento de coordenadas
    coordinateLayout._container.classList.add('context-menu-popup');
}

// funcoes para dar zoom 
function zoomIn() {
    ZoomInOut('in');
}

function zoomOut() {
    ZoomInOut('out');
}

// funcao para mudar a rota da página
const redirectGraph = () => {
    window.location.href = '/grafico';
};


onMounted(() => {
    // Objeto com as configurações do Leaflet para criação do objeto map
    let configs = ConfigsLeaflet(configsMapa);
    
    // Cria o objeto map do Leaflet
    map = L.map('map', configs);

    // Adiciona a função "SetaVisaoInicial" de resetar a visão inicial do mapa
    FuncaoVisaoInicial(map, configsMapa.visaoInicial.x, configsMapa.visaoInicial.y, configsMapa.visaoInicial.z);

    // Seta a visualização inicial do mapa
    SetaVisaoInicial();

    // Adiciona as funções de zoom do mapa
    FuncoesZoom(map);

    // Adiciona a função "MapaInformacoes" para mostrar as informações do mapa
    FuncaoMapaInformacoes(map);
    
    // Adiciona as coordenadas do mouse no canto inferior direito do mapa
    configsMapa.funcionalidades.coordenadasMouse ? AdicionarCoordenadasMouse(map, configsMapa.configuracoesLeaflet) : configsMapa.funcionalidades.coordenadasMouse;

    // Desabilitar o clique-duplo para função de zoom (nativa do Leaflet)
    map.doubleClickZoom.disable();

    // Adiciona escala ao mapa caso configsMapa.funcionalidades.escala seja true
    configsMapa.funcionalidades.escala ? AdicionarEscala(map, 'metric') : configsMapa.funcionalidades.escala;

    // Adiciona atribuições (fonte de dados) ao mapa    
    configsMapa.funcionalidades.atribuicoes ? AdicionaAttribution(map, configsMapa.configuracoesLeaflet.atribuicaoPrefixo) : map.attributionControl.remove();
    
    // Desativa o clique do botão direito para evitar menu de contexto do navegador e cria o menu de contexto (clique direito com coordenadas)
    configsMapa.funcionalidades.menuContexto ? CriaMenuContexto(map, configsMapa.configuracoesLeaflet) : configsMapa.funcionalidades.menuContexto;

    // Adiciona o basemap padrão do mapa (TileLayers)
    AdicionaBasemapPadrao(map, rasters);

    // Adiciona os overlays (TileLayers)
    AdicionaOverlaysPadrao(map, rasters);

    window.ToggleRaster = function (nomeRaster) {
        rasters.forEach(raster => raster.nome == nomeRaster ? ToggleRasterTile(map, raster) : null);
    };
});
</script>

<template>
    <div id="map" class="z-[5] h-[calc(100vh)] max-h-[calc(100vh)]"></div>
        <div id="zoom-buttons">
            <button @click="zoomIn" class="zoom-button zoom-in">Zoom In</button>
            <button @click="zoomOut" class="zoom-button zoom-out">Zoom Out</button>
        </div>
        <div id="map-controls">
            <button @click="ToggleRasterTile()" class="toggle-map-button">Alternar Mapa</button>
        </div>
        <button class="page-button" @click="redirectGraph">Ir para Gráficos</button>
</template>