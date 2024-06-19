<script setup>
import { ref, reactive, onMounted, computed } from "vue";
import Alerta from "./components/Alerta.vue";
import Spinner from "./components/Spinner.vue";

const monedas = ref([
  { codigo: "COP", texto: "Peso Colombiano" },
  { codigo: "USD", texto: "Dolar de Estados Unidos" },
  { codigo: "MXN", texto: "Peso Mexicano" },
  { codigo: "EUR", texto: "Euro" },
  { codigo: "GBP", texto: "Libra Esterlina" },
]);

const criptomonedas = ref([]);
const error = ref("");
const cargando = ref(false);
const cotizar = reactive({
  moneda: "",
  criptomoneda: "",
});

const cotizacion = ref({});

onMounted(() => {
  const url =
    "https://min-api.cryptocompare.com/data/top/mktcapfull?limit=20&tsym=USD";
  fetch(url)
    .then((respuesta) => respuesta.json())
    .then((data) => (criptomonedas.value = data.Data))
    .catch((error) => console.log(error));
});

const cotizarCripto = () => {
  // validar selects
  if (Object.values(cotizar).includes("")) {
    error.value = "Todos los campos son obligatorios";
    return;
  }
  error.value = "";
  obtenerCotizacion();
};
const obtenerCotizacion = async () => {
  cargando.value = true;
  cotizacion.value = {};
  const { moneda, criptomoneda } = cotizar;
  const url = `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${criptomoneda}&tsyms=${moneda}`;
  const respuesta = await fetch(url);
  const data = await respuesta.json();
  cotizacion.value = data.DISPLAY[criptomoneda][moneda];

  cargando.value = false;
};

const mostrarResultado = computed(() => {
  return Object.values(cotizacion.value).length > 0;
});
</script>

<template>
  <div class="contenedor">
    <h1 class="titulo">cotizador de <span> Criptomonedas</span></h1>
    <div class="contenido">
      <Alerta v-if="error">
        {{ error }}
      </Alerta>

      <form class="formulario" @submit.prevent="cotizarCripto">
        <div class="campo">
          <label for="moneda">Moneda:</label>
          <select id="moneda" v-model="cotizar.moneda">
            <option value="" selected disabled>Seleccione una moneda</option>
            <option
              v-for="moneda in monedas"
              :key="moneda.codigo"
              :value="moneda.codigo"
            >
              {{ moneda.texto }}
            </option>
          </select>
        </div>

        <div class="campo">
          <label for="cripto">CriptoMoneda:</label>
          <select id="cripto" v-model="cotizar.criptomoneda">
            <option value="" disabled>Seleccione una Criptomoneda</option>
            <option
              v-for="criptomoneda in criptomonedas"
              :key="criptomoneda.id"
              :value="criptomoneda.CoinInfo.Name"
            >
              {{ criptomoneda.CoinInfo.FullName }}
            </option>
          </select>
        </div>

        <input type="submit" value="Cotizar" />
      </form>
      <Spinner v-if="cargando" />
      <div v-if="mostrarResultado" class="contenedor-resultado">
        <h2>Cotización</h2>
        <div class="resultado">
          <img
            :src="'https://cryptocompare.com/' + cotizacion.IMAGEURL"
            alt="imagen cripto"
          />
          <div>
            <p>
              El precio es de: <span>{{ cotizacion.PRICE }}</span>
            </p>
            <p>
              Precio más alto del día: <span>{{ cotizacion.HIGHDAY }}</span>
            </p>
            <p>
              Precio más bajo del día: <span>{{ cotizacion.LOWDAY }}</span>
            </p>
            <p>
              Variación en 24 Horas: %<span>{{
                cotizacion.CHANGEPCT24HOUR
              }}</span>
            </p>
            <p>
              Última actualización: <span>{{ cotizacion.LASTUPDATE }}</span>
            </p>
          </div>
        </div>
      </div>
    </div>
    <div class="info_flex">
     
      <div class="textoapi">
        consumo de la api:
        <span class="enlace"
          ><a target="_blank" href="https://min-api.cryptocompare.com/"
            >https://min-api.cryptocompare.com/</a
          ></span
        >
      </div>
    </div>
  </div>
</template>

