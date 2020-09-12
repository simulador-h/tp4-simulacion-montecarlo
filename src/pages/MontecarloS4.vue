<template>
  <q-page>
    <q-tabs
      v-model="activeTab"
      align="justify"
      active-color="primary"
      indicator-color="primary"
      class="text-grey"
    >
      <q-tab name="parameters" label="Parámetros" />
      <q-tab name="simulation" label="Simulación" />
    </q-tabs>

    <q-tab-panels v-model="activeTab" animated keep-alive>
      <q-tab-panel name="parameters">
        <parameters-s-4 :parameters="parameters" @submit="save" @reset="reload" />
      </q-tab-panel>
      <q-tab-panel name="simulation">
        <simulation-s-4 :parameters="parameters" />
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script lang="ts">
  import {
    defineComponent,
    reactive,
    toRefs,
  } from '@vue/composition-api';

  import ParametersS4, { IParametersS4 } from 'components/ParametersS4.vue';
  import SimulationS4 from 'components/SimulationS4.vue';

  function useMontecarloS4() {
    const state = reactive({
      activeTab: 'parameters',
      parameters: {
        pCompra: {
          cantidadFrascos: 2,
          tamañoFrascos: 170,
          intervaloCompra: 2,
        },
        pCosto: {
          costoCompra: 250,
          costoOportunidad: 1,
        },
        pVenta: {
          precioVenta: 1.5,
        },
        pStock: {
          capacidadMaxima: 10,
        },
      },
      save: (parameters: IParametersS4) => {
        /* eslint-disable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */
        const {
          pCompra: { cantidadFrascos, tamañoFrascos, intervaloCompra },
          pCosto: { costoCompra, costoOportunidad },
          pVenta: { precioVenta },
          pStock: { capacidadMaxima },
        } = parameters;

        state.parameters.pCompra.cantidadFrascos = cantidadFrascos;
        state.parameters.pCompra.tamañoFrascos = tamañoFrascos;
        state.parameters.pCompra.intervaloCompra = intervaloCompra;
        state.parameters.pCosto.costoCompra = costoCompra;
        state.parameters.pCosto.costoOportunidad = costoOportunidad;
        state.parameters.pVenta.precioVenta = precioVenta;
        state.parameters.pStock.capacidadMaxima = capacidadMaxima;
        /* eslint-enable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */
      },
      reload: (parameters: IParametersS4) => {
        /* eslint-disable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */
        const {
          pCompra: { cantidadFrascos, tamañoFrascos, intervaloCompra },
          pCosto: { costoCompra, costoOportunidad },
          pVenta: { precioVenta },
          pStock: { capacidadMaxima },
        } = state.parameters;

        parameters.pCompra.cantidadFrascos = cantidadFrascos;
        parameters.pCompra.tamañoFrascos = tamañoFrascos;
        parameters.pCompra.intervaloCompra = intervaloCompra;
        parameters.pCosto.costoCompra = costoCompra;
        parameters.pCosto.costoOportunidad = costoOportunidad;
        parameters.pVenta.precioVenta = precioVenta;
        parameters.pStock.capacidadMaxima = capacidadMaxima;
        /* eslint-enable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */
      },
    } as {
      activeTab: string
      parameters: IParametersS4
    });

    return toRefs(state);
  }

  export default defineComponent({
    name: 'MontecarloS4',
    components: { ParametersS4, SimulationS4 },
    setup() {
      return useMontecarloS4();
    },
  });
</script>
