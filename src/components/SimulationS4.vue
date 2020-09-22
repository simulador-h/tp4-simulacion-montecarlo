<template>
  <q-page padding>
    <q-form @submit.prevent="onSubmit">
      <div class="row q-col-gutter-md">
        <div class="col-5">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Parámetros de simulación
              </span>
              <q-icon name="settings" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model.number="iteraciones"
                label="Duración de la simulación *"
                suffix="días"
                type="number"
                outlined
                lazy-rules
                :rules="[ val => (val && val > 0) || 'Requerido | Mayor a 0' ]"
              />

              <q-item tag="label">
                <q-item-section>
                  <q-item-label>Generador Aleatorio</q-item-label>
                  <q-item-label caption>
                    {{ generador === 'native' ? 'Nativo' : 'Congruencial' }}
                  </q-item-label>
                </q-item-section>
                <q-item-section avatar>
                  <q-toggle
                    v-model="generador"
                    true-value="native"
                    false-value="lcg"
                    color="pink"
                  />
                </q-item-section>
              </q-item>
            </q-card-section>
          </q-card>
        </div>

        <div class="col-5">
          <q-card flat bordered class="full-height">
            <q-card-section class="row items-center justify-between">
              <span class="text-uppercase text-caption text-primary">
                Parámetros de muestreo
              </span>
              <q-icon name="preview" class="text-primary" />
            </q-card-section>

            <q-separator inset />

            <q-card-section>
              <q-input
                v-model="condicionMuestreo"
                label="Condición de muestreo *"
                type="textarea"
                outlined
                lazy-rules
                :rules="[ validators.validJS ]"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col row justify-end">
          <q-btn label="Simular" type="submit" color="primary" class="full-width" />
        </div>
      </div>
    </q-form>

    <q-table
      title="Cibercafé"
      class="q-mt-md"
      flat dense bordered
      row-key="día"
      :columns="columnas"
      :visible-columns="columnasVisibles"
      :data="vectores"
      :loading="loading"
      :pagination="{
        sortBy: 'día',
        rowsPerPage: 50,
      }"
    >
      <template #top="props">
        <div class="q-table__title">
          Cibercafé
        </div>
        <q-space />
        <q-select
          v-model="columnasVisibles"
          class="col-2"
          label="Columnas"
          :options="columnas"
          :option-disable="({ required }) => required"
          :display-value="columnasVisibles.length"
          option-value="name"
          borderless
          multiple
          dense
          options-dense
          emit-value
          map-options
        >
          <template #prepend>
            <q-icon name="view_week" />
          </template>
        </q-select>
        <q-btn
          class="q-ml-md"
          flat round dense
          :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'"
          @click="props.toggleFullscreen"
        />
      </template>

      <template #loading>
        <q-inner-loading showing color="primary" />
      </template>

      <template #no-data>
        <div class="row flex-center full-width" style="height: 3rem">
          <q-icon size="2em" name="pest_control_rodent" />
        </div>
      </template>
    </q-table>
  </q-page>
</template>

<style lang="scss" scoped>
  .q-card .q-card__section .q-icon {
    font-size: 1.25rem;
  }

  /deep/ .q-table {
    thead tr:first-child th:first-child {
      color: white;
      background-color: $primary;
    }

    td:first-child {
      background-color: #f5f5f5;
    }

    th:first-child,
    td:first-child {
      position: sticky;
      left: 0;
      min-width: 3.5rem;
      z-index: 1;
    }
  }
</style>

<script lang="ts">
  import {
    defineComponent,
    reactive,
    toRefs,
    PropType,
  } from '@vue/composition-api';

  import { IParametersS4 } from './ParametersS4.vue';

  export interface VectorEstado {
    día: number
    stock: {
      disponible: {
        gramos: number
        frascos: number
      }
      remanente: {
        gramos: number
        frascos: number
      }
      descartado: {
        gramos: number
        frascos: number
      }
      final: {
        gramos: number
        frascos: number
      }
    }
    demanda: {
      mañana: number
      tarde: number
      satisfecha: number
      insatisfecha: number
    }
    pedido: {
      demora: number
      llegada: number
    }
    ganancia: {
      venta: number
    }
    costo: {
      compra: number
      oportunidad: number
    }
    balance: {
      contribución: number
    }
    estadístico: {
      stock: {
        descartadoPromedio: {
          gramos: number
          frascos: number
        }
        finalPromedio: {
          gramos: number
          frascos: number
        }
        finalFrecuencia: {
          gramos: {
            0?: number
          }
          frascos: {
            [key: number]: number
          }
        }
        finalPorcentajes: {
          gramos: {
            0?: number
          }
          frascos: {
            [key: number]: number
          }
        }
      }
      demanda: {
        insatisfechaPromedio: number
        insatisfechaAcumulada: {
          días: number
        }
        insatisfechaPorcentaje: number
      }
      ganancia: {
        ventaPromedio: number
      }
      costo: {
        compraPromedio: number
        oportunidadPromedio: number,
      }
      balance: {
        contribuciónPromedio: number
      }
    }
  }

  function* LCG(multiplier: number, increment: number, modulus: number, seed: number) {
    let x = seed;

    while (true) {
      x = ((multiplier * x) + increment) % modulus;
      yield x / modulus;
    }
  }

  function* MWC1616() {
    while (true) {
      yield Math.random();
    }
  }

  const nn = (v: number) => Number(
    v.toFixed(0),
  );

  const n3 = (v: number) => Number(
    v.toFixed(3),
  );

  const np = (v: number) => (
    (v % 1) ? `${Math.floor(v)}+` : v
  );

  const $ = (v: number) => (
    `$ ${v.toFixed(0)}`
  );

  const percent = (v: number) => `${
    Number(
      (v * 100).toFixed(0),
    )
  } %`;

  function useSimulationS4({ parameters }: { parameters: IParametersS4 }) {
    const state = reactive({
      iteraciones: 100000,
      generador: 'native',
      condicionMuestreo: 'n <= 25 || !(n % 10000)',

      columnas: [
        {
          name: 'día',
          label: 'Día',
          field: (row: VectorEstado) => row.día,
          align: 'right',
          required: true,
        },
        {
          name: 'stock-disponible-g',
          label: 'Stock Disponible (g)',
          field: (row: VectorEstado) => row.stock.disponible.gramos,
          align: 'right',
          format: nn,
        },
        {
          name: 'stock-disponible-fr',
          label: 'Stock Disponible (fr)',
          field: (row: VectorEstado) => row.stock.disponible.frascos,
          format: np,
          align: 'right',
        },
        {
          name: 'stock-remanente-g',
          label: 'Stock Remanente (g)',
          field: (row: VectorEstado) => row.stock.remanente.gramos,
          format: nn,
          align: 'right',
        },
        {
          name: 'stock-remanente-fr',
          label: 'Stock Remanente (fr)',
          field: (row: VectorEstado) => row.stock.remanente.frascos,
          format: np,
          align: 'right',
        },
        {
          name: 'stock-descartado-g',
          label: 'Stock Descartado (g)',
          field: (row: VectorEstado) => row.stock.descartado.gramos,
          format: nn,
          align: 'right',
        },
        {
          name: 'stock-descartado-fr',
          label: 'Stock Descartado (fr)',
          field: (row: VectorEstado) => row.stock.descartado.frascos,
          format: np,
          align: 'right',
        },
        {
          name: 'stock-final-g',
          label: 'Stock Final (g)',
          field: (row: VectorEstado) => row.stock.final.gramos,
          format: nn,
          align: 'right',
        },
        {
          name: 'stock-final-fr',
          label: 'Stock Final (fr)',
          field: (row: VectorEstado) => row.stock.final.frascos,
          format: np,
          align: 'right',
        },
        {
          name: 'demanda-mañana',
          label: 'Demanda Mañana (g)',
          field: (row: VectorEstado) => row.demanda.mañana,
          format: nn,
          align: 'right',
        },
        {
          name: 'demanda-tarde',
          label: 'Demanda Tarde (g)',
          field: (row: VectorEstado) => row.demanda.tarde,
          format: nn,
          align: 'right',
        },
        {
          name: 'demanda-satisfecha',
          label: 'Demanda Satisfecha (g)',
          field: (row: VectorEstado) => row.demanda.satisfecha,
          format: nn,
          align: 'right',
        },
        {
          name: 'demanda-insatisfecha',
          label: 'Demanda Insatisfecha (g)',
          field: (row: VectorEstado) => row.demanda.insatisfecha,
          format: nn,
          align: 'right',
        },
        {
          name: 'pedido-demora',
          label: 'Demora Pedido (días)',
          field: (row: VectorEstado) => row.pedido.demora,
          format: (v) => (
            (typeof v === 'number') ? v : '-'
          ),
          align: 'right',
        },
        {
          name: 'pedido-llegada',
          label: 'Llegada Pedido',
          field: (row: VectorEstado) => row.pedido.llegada,
          format: (v: number) => `Día ${v}`,
          align: 'right',
        },
        {
          name: 'ganancia-venta',
          label: 'Ganancia Venta',
          field: (row: VectorEstado) => row.ganancia.venta,
          format: $,
          align: 'right',
        },
        {
          name: 'costo-compra',
          label: 'Costo Compra',
          field: (row: VectorEstado) => row.costo.compra,
          format: $,
          align: 'right',
        },
        {
          name: 'costo-oportunidad',
          label: 'Costo Oportunidad',
          field: (row: VectorEstado) => row.costo.oportunidad,
          format: $,
          align: 'right',
        },
        {
          name: 'balance-contribución',
          label: 'Márgen Contribución',
          field: (row: VectorEstado) => row.balance.contribución,
          format: $,
          align: 'right',
        },
        {
          name: 'estadístico-stock-descartado-promedio-g',
          label: 'Stock Descartado Promedio (g)',
          field: (row: VectorEstado) => row.estadístico.stock.descartadoPromedio.gramos,
          format: nn,
          align: 'right',
        },
        {
          name: 'estadístico-stock-descartado-promedio-fr',
          label: 'Stock Descartado Promedio (fr)',
          field: (row: VectorEstado) => row.estadístico.stock.descartadoPromedio.frascos,
          format: n3,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-promedio-g',
          label: 'Stock Final Promedio (g)',
          field: (row: VectorEstado) => row.estadístico.stock.finalPromedio.gramos,
          format: nn,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-promedio-fr',
          label: 'Stock Final Promedio (fr)',
          field: (row: VectorEstado) => row.estadístico.stock.finalPromedio.frascos,
          format: n3,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-frecuencia-2',
          label: 'Stock Final <= 2 (fr)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalFrecuencia.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) <= 2 ? sum + v : sum
            ), 0),
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-porcentaje-2',
          label: 'Stock Final <= 2 (%)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalPorcentajes.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) <= 2 ? sum + v : sum
            ), 0),
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-frecuencia-2-5',
          label: 'Stock Final (2;5] (fr)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalFrecuencia.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 2 && Number(k) <= 5 ? sum + v : sum
            ), 0),
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-porcentaje-2-5',
          label: 'Stock Final (2;5] (%)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalPorcentajes.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 2 && Number(k) <= 5 ? sum + v : sum
            ), 0),
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-frecuencia-5-8',
          label: 'Stock Final (5;8] (fr)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalFrecuencia.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 5 && Number(k) <= 8 ? sum + v : sum
            ), 0),
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-porcentaje-5-8',
          label: 'Stock Final (5;8] (%)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalPorcentajes.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 5 && Number(k) <= 8 ? sum + v : sum
            ), 0),
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-frecuencia-8',
          label: 'Stock Final > 8 (fr)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalFrecuencia.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 8 ? sum + v : sum
            ), 0),
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-porcentaje-8',
          label: 'Stock Final > 8 (%)',
          field: (row: VectorEstado) => Object
            .entries(row.estadístico.stock.finalPorcentajes.frascos)
            .reduce((sum, [k, v]) => (
              Number(k) > 8 ? sum + v : sum
            ), 0),
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-stock-final-porcentaje-stock-0',
          label: 'Stock Final Positivo (%)',
          field: (row: VectorEstado) => 1 - (row.estadístico.stock.finalPorcentajes.gramos[0] || 0),
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-demanda-insatisfecha-promedio',
          label: 'Demanda Insatisfecha Promedio (g)',
          field: (row: VectorEstado) => row.estadístico.demanda.insatisfechaPromedio,
          format: nn,
          align: 'right',
        },
        {
          name: 'estadístico-demanda-insatisfecha-acumulada',
          label: 'Demanda Insatisfecha (días)',
          field: (row: VectorEstado) => row.estadístico.demanda.insatisfechaAcumulada.días,
          align: 'right',
        },
        {
          name: 'estadístico-demanda-insatisfecha-porcentaje',
          label: 'Demanda Insatisfecha (%)(días)',
          field: (row: VectorEstado) => row.estadístico.demanda.insatisfechaPorcentaje,
          format: percent,
          align: 'right',
        },
        {
          name: 'estadístico-ganancia-venta-promedio',
          label: 'Ganancia Venta Promedio',
          field: (row: VectorEstado) => row.estadístico.ganancia.ventaPromedio,
          format: $,
          align: 'right',
        },
        {
          name: 'estadístico-costo-compra-promedio',
          label: 'Costo Compra Promedio',
          field: (row: VectorEstado) => row.estadístico.costo.compraPromedio,
          format: $,
          align: 'right',
        },
        {
          name: 'estadístico-costo-oportunidad-promedio',
          label: 'Costo Oportunidad Promedio',
          field: (row: VectorEstado) => row.estadístico.costo.oportunidadPromedio,
          format: $,
          align: 'right',
        },
        {
          name: 'estadístico-balance-contribución-promedio',
          label: 'Márgen Contribución Promedio',
          field: (row: VectorEstado) => row.estadístico.balance.contribuciónPromedio,
          format: $,
          align: 'right',
        },
      ],
      columnasVisibles: [
        'día',
        'stock-disponible-g', 'stock-remanente-g', 'stock-descartado-g', 'stock-final-g',
        'demanda-mañana', 'demanda-tarde', 'demanda-satisfecha', 'demanda-insatisfecha',
        'pedido-demora', 'pedido-llegada',
        'ganancia-venta', 'costo-compra', 'costo-oportunidad', 'balance-contribución',
        'estadístico-stock-descartado-promedio-g', 'estadístico-stock-final-promedio-g',
        'estadístico-demanda-insatisfecha-promedio', 'estadístico-demanda-insatisfecha-porcentaje',
        'estadístico-ganancia-venta-promedio',
        'estadístico-costo-compra-promedio', 'estadístico-costo-oportunidad-promedio',
        'estadístico-balance-contribución-promedio',
      ],

      vectores: <VectorEstado[]>[],
      ultimoDíaVector: -1,

      loading: false,

      validators: {
        validJS: (val: string) => {
          try {
            // eslint-disable-next-line @typescript-eslint/no-implied-eval, no-new-func, no-new
            new Function('n', `return (${val});`); return true;
          }
          catch (e) {
            return 'No es una expresión JavaScript válida.';
          }
        },
      },
      onSubmit: () => {
        const generator = (state.generador === 'lcg') ? LCG(13, 43, 101, 37) : MWC1616();

        // eslint-disable-next-line @typescript-eslint/no-implied-eval, no-new-func
        const samplingEvaluator = new Function('n', `return (${state.condicionMuestreo});`);

        state.vectores = [];
        state.ultimoDíaVector = -1;
        state.loading = true;

        /* eslint-disable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */
        const {
          pCompra: { cantidadFrascos, tamañoFrascos, intervaloCompra },
          pCosto: { costoCompra, costoOportunidad },
          pVenta: { precioVenta },
          pStock: { capacidadMaxima },
        } = parameters;
        /* eslint-enable @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-member-access */

        function obtenerDemoraPedidoAleatoria() {
          const r = generator.next().value as number;

          if (r < 0.5) {
            return 0;
          }

          if (r < 0.75) {
            return 1;
          }

          return 2;
        }

        function obtenerDemandaMatutinaAleatoria() {
          const r = generator.next().value as number;

          if (r < 0.5) {
            return 50;
          }

          const r1 = generator.next().value as number;
          const r2 = generator.next().value as number;

          const random = Math.sqrt(-2 * Math.log(1 - r1)) * Math.cos(2 * Math.PI * r2);
          return 75 + random * 15;
        }

        function obtenerDemandaVespertinaAleatoria() {
          const r = generator.next().value as number;
          return (-70) * Math.log(1 - r);
        }

        let dI: VectorEstado = {
          día: 0,
          stock: {
            disponible: { gramos: 0, frascos: 0 },
            remanente: { gramos: 0, frascos: 0 },
            descartado: { gramos: 0, frascos: 0 },
            final: { gramos: 0, frascos: 0 },
          },
          demanda: {
            mañana: 0,
            tarde: 0,
            satisfecha: 0,
            insatisfecha: 0,
          },
          pedido: {
            demora: 1,
            llegada: 1,
          },
          ganancia: {
            venta: 0,
          },
          costo: {
            compra: 0,
            oportunidad: 0,
          },
          balance: {
            contribución: 0,
          },
          estadístico: {
            stock: {
              descartadoPromedio: {
                gramos: 0,
                frascos: 0,
              },
              finalPromedio: {
                gramos: 0,
                frascos: 0,
              },
              finalFrecuencia: {
                gramos: {},
                frascos: {},
              },
              finalPorcentajes: {
                gramos: {},
                frascos: {},
              },
            },
            demanda: {
              insatisfechaPromedio: 0,
              insatisfechaAcumulada: {
                días: 0,
              },
              insatisfechaPorcentaje: 0,
            },
            ganancia: {
              ventaPromedio: 0,
            },
            costo: {
              compraPromedio: 0,
              oportunidadPromedio: 0,
            },
            balance: {
              contribuciónPromedio: 0,
            },
          },
        };

        for (let n = 1; n <= state.iteraciones; n++) {
          const dN = {
            día: n,
            stock: {
              disponible: { gramos: 0, frascos: 0 },
              remanente: { gramos: 0, frascos: 0 },
              descartado: { gramos: 0, frascos: 0 },
              final: { gramos: 0, frascos: 0 },
            },
            demanda: {
              mañana: 0,
              tarde: 0,
              satisfecha: 0,
              insatisfecha: 0,
            },
            pedido: {
              llegada: 0,
            },
            ganancia: {
              venta: 0,
            },
            costo: {
              compra: 0,
              oportunidad: 0,
            },
            balance: {
              contribución: 0,
            },
            estadístico: {
              stock: {
                descartadoPromedio: { gramos: 0, frascos: 0 },
                finalPromedio: { gramos: 0, frascos: 0 },
                finalFrecuencia: { gramos: {}, frascos: {} },
                finalPorcentajes: { gramos: {}, frascos: {} },
              },
              demanda: {
                insatisfechaPromedio: 0,
                insatisfechaAcumulada: { días: 0 },
                insatisfechaPorcentaje: 0,
              },
              ganancia: {
                ventaPromedio: 0,
              },
              costo: {
                compraPromedio: 0,
                oportunidadPromedio: 0,
              },
              balance: {
                contribuciónPromedio: 0,
              },
            },
          } as VectorEstado;

          let stockEntrante = (n === dI.pedido.llegada) ? cantidadFrascos * tamañoFrascos : 0;

          if (n % intervaloCompra) {
            dN.pedido.llegada = dI.pedido.llegada;
          }
          else {
            dN.pedido.demora = obtenerDemoraPedidoAleatoria();
            dN.pedido.llegada = n + dN.pedido.demora;
            if (!dN.pedido.demora) {
              stockEntrante += cantidadFrascos * tamañoFrascos;
            }
          }

          dN.stock.disponible.gramos = dI.stock.final.gramos + stockEntrante;
          dN.stock.disponible.frascos = dN.stock.disponible.gramos / tamañoFrascos;

          dN.demanda.mañana = obtenerDemandaMatutinaAleatoria();
          dN.demanda.tarde = obtenerDemandaVespertinaAleatoria();

          const demandaTotal = dN.demanda.mañana + dN.demanda.tarde;
          dN.demanda.satisfecha = Math.min(demandaTotal, dN.stock.disponible.gramos);
          dN.demanda.insatisfecha = demandaTotal - dN.demanda.satisfecha;

          dN.stock.remanente.gramos = dN.stock.disponible.gramos - dN.demanda.satisfecha;
          dN.stock.remanente.frascos = dN.stock.remanente.gramos / tamañoFrascos;

          dN.stock.final.gramos = Math.min((capacidadMaxima * tamañoFrascos), dN.stock.remanente.gramos);
          dN.stock.final.frascos = dN.stock.final.gramos / tamañoFrascos;

          dN.stock.descartado.gramos = dN.stock.remanente.gramos - dN.stock.final.gramos;
          dN.stock.descartado.frascos = dN.stock.descartado.gramos / tamañoFrascos;

          dN.ganancia.venta = dN.demanda.satisfecha * precioVenta;

          dN.costo.compra = !(n % intervaloCompra) ? costoCompra * cantidadFrascos : 0;
          dN.costo.oportunidad = dN.demanda.insatisfecha * costoOportunidad;

          dN.balance.contribución = dN.ganancia.venta - dN.costo.compra;

          dN.estadístico.stock.descartadoPromedio.gramos = (
            (dI.estadístico.stock.descartadoPromedio.gramos * dI.día) + dN.stock.descartado.gramos
          ) / n;
          dN.estadístico.stock.descartadoPromedio.frascos = (
            dN.estadístico.stock.descartadoPromedio.gramos
          ) / tamañoFrascos;

          dN.estadístico.stock.finalPromedio.gramos = (
            (dI.estadístico.stock.finalPromedio.gramos * dI.día) + dN.stock.final.gramos
          ) / n;
          dN.estadístico.stock.finalPromedio.frascos = (
            dN.estadístico.stock.finalPromedio.gramos
          ) / tamañoFrascos;

          const stockCero = dN.stock.final.gramos === 0;
          dN.estadístico.stock.finalFrecuencia.gramos[0] = dI.estadístico.stock.finalFrecuencia.gramos[0] || 0;
          if (stockCero) {
            dN.estadístico.stock.finalFrecuencia.gramos[0] += 1;
          }
          dN.estadístico.stock.finalPorcentajes.gramos[0] = dN.estadístico.stock.finalFrecuencia.gramos[0] / n;

          const frascosLlenos = Math.floor(dN.stock.final.frascos);
          dN.estadístico.stock.finalFrecuencia.frascos = { ...dI.estadístico.stock.finalFrecuencia.frascos };
          const fI = dI.estadístico.stock.finalFrecuencia.frascos[frascosLlenos] || 0;
          dN.estadístico.stock.finalFrecuencia.frascos[frascosLlenos] = fI + 1;
          dN.estadístico.stock.finalPorcentajes.frascos = { ...dN.estadístico.stock.finalFrecuencia.frascos };

          // eslint-disable-next-line guard-for-in, no-restricted-syntax
          for (const k in dN.estadístico.stock.finalPorcentajes.frascos) {
            dN.estadístico.stock.finalPorcentajes.frascos[k] /= n;
          }

          dN.estadístico.demanda.insatisfechaPromedio = (
            (dI.estadístico.demanda.insatisfechaPromedio * dI.día) + dN.demanda.insatisfecha
          ) / n;

          dN.estadístico.demanda.insatisfechaAcumulada.días = dI.estadístico.demanda.insatisfechaAcumulada.días;
          if (dN.demanda.insatisfecha) {
            dN.estadístico.demanda.insatisfechaAcumulada.días += 1;
          }

          dN.estadístico.demanda.insatisfechaPorcentaje = dN.estadístico.demanda.insatisfechaAcumulada.días / n;

          dN.estadístico.ganancia.ventaPromedio = (
            (dI.estadístico.ganancia.ventaPromedio * dI.día) + dN.ganancia.venta
          ) / n;

          dN.estadístico.costo.compraPromedio = (
            (dI.estadístico.costo.compraPromedio * dI.día) + dN.costo.compra
          ) / n;

          dN.estadístico.costo.oportunidadPromedio = (
            (dI.estadístico.costo.oportunidadPromedio * dI.día) + dN.costo.oportunidad
          ) / n;

          dN.estadístico.balance.contribuciónPromedio = (
            (dI.estadístico.balance.contribuciónPromedio * dI.día) + dN.balance.contribución
          ) / n;

          if (samplingEvaluator(n)) {
            if (state.ultimoDíaVector !== dI.día) {
              state.vectores.push({ ...dI });
            }
            state.vectores.push({ ...dN });
            state.ultimoDíaVector = n;
          }

          dI = dN;

          state.loading = false;
        }
      },
    });

    return toRefs(state);
  }

  export default defineComponent({
    name: 'SimulationS4',
    components: {},
    props: {
      parameters: {
        required: true,
        type: Object as PropType<IParametersS4>,
      },
    },
    setup(props) {
      return useSimulationS4(props);
    },
  });
</script>
