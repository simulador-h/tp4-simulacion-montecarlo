<template>
  <q-page padding>
    <q-form
      @submit.prevent="onSubmit"
      @reset="onReset"
    >
      <div class="row q-col-gutter-md">
        <div class="col-12 col-sm-6">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Política de compras
              </span>
              <q-icon name="shopping_bag" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model.number="pCompra.cantidadFrascos"
                label="Cantidad de frascos *"
                suffix="frascos"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />

              <q-input
                v-model.number="pCompra.tamañoFrascos"
                label="Tamaño de frascos *"
                suffix="gramos"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />

              <q-input
                v-model.number="pCompra.intervaloCompra"
                label="Intervalo de compra *"
                suffix="días"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col-12 col-sm-6">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Política de costos
              </span>
              <q-icon name="money_off" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model.number="pCosto.costoCompra"
                label="Costo de compra *"
                prefix="$"
                suffix="/frasco"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />

              <q-input
                v-model.number="pCosto.costoOportunidad"
                label="Costo de oportunidad *"
                prefix="$"
                suffix="/gramo"
                type="text"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col-12 col-sm-6">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Política de ventas
              </span>
              <q-icon name="local_cafe" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model.number="pVenta.precioVenta"
                label="Precio de venta *"
                prefix="$"
                suffix="/gramo"
                type="text"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col-12 col-sm-6">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Política de stock
              </span>
              <q-icon name="storage" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model.number="pStock.capacidadMaxima"
                label="Capacidad máxima *"
                suffix="frascos"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col row justify-end">
          <q-btn label="Guardar" type="submit" color="primary" />
          <q-btn label="Restablecer" type="reset" color="primary" flat class="q-ml-sm" />
        </div>
      </div>
    </q-form>
  </q-page>
</template>

<style lang="scss" scoped>
  .q-card .q-card__section .q-icon {
    font-size: 1.25rem;
  }
</style>

<script lang="ts">
  import {
    defineComponent,
    reactive,
    PropType,
  } from '@vue/composition-api';

  export interface IParametersS4 {
    pCompra: {
      cantidadFrascos: number
      tamañoFrascos: number
      intervaloCompra: number
    }
    pCosto: {
      costoCompra: number
      costoOportunidad: number
    }
    pVenta: {
      precioVenta: number
    }
    pStock: {
      capacidadMaxima: number
    }
  }

  export default defineComponent({
    name: 'ParametersS4',
    components: {},
    props: {
      parameters: {
        required: true,
        type: Object as PropType<IParametersS4>,
      },
    },
    // eslint-disable-next-line vue/no-setup-props-destructure
    setup({ parameters }, { emit }) {
      const pCompra = reactive({ ...parameters.pCompra });
      const pCosto = reactive({ ...parameters.pCosto });
      const pVenta = reactive({ ...parameters.pVenta });
      const pStock = reactive({ ...parameters.pStock });

      const onSubmit = () => {
        emit('submit', {
          pCompra,
          pCosto,
          pVenta,
          pStock,
        } as IParametersS4);
      };

      const onReset = () => {
        emit('reset', {
          pCompra,
          pCosto,
          pVenta,
          pStock,
        } as IParametersS4);
      };

      return {
        pCompra,
        pCosto,
        pVenta,
        pStock,
        onSubmit,
        onReset,
      };
    },
  });
</script>
