<template>
  <div class="pos">
    <!-- ================ ZONA MENÚ — PERSONA A (AGREGAR) ================ -->
    <section class="card">
      <h2>🍔 Menú — toca para agregar</h2>

      <!-- Persona A:
           Cada tarjeta del menú dispara @click="agregar(p)",
           que añade el producto al pedido o incrementa su cantidad -->
      <div
        class="food-card"
        v-for="p in menu"
        :key="p.id"
        @click="agregar(p)"
      >
        <div class="emoji">{{ p.emoji }}</div>
        <div class="nom">{{ p.nombre }}</div>
        <div class="pre">${{ p.precio.toLocaleString('es-CL') }}</div>
      </div>
    </section>

    <!-- ================ ZONA TICKET — A + B (PEDIDO COMPLETO) ================ -->
    <section class="card">
      <h2>🧾 Ticket del pedido</h2>

      <div class="ticket">
        <h4>Pedido</h4>

        <!-- B · Req 7: Cobrar sin recargar (@submit.prevent) -->
        <form @submit.prevent="cobrar">
          <!-- A · Req 3: Estado vacío del ticket -->
          <div v-if="pedido.length === 0" class="vacio">
            Aún no hay productos
          </div>

          <!-- A · Req 1 + B · Req 4, 5: Lista del pedido editable -->
          <div v-else>
            <div class="item" v-for="item in pedido" :key="item.id">
              <div>
                <span>{{ item.emoji }}</span>
                <span> {{ item.nombre }} </span>
              </div>

              <div class="qty">
                <!-- B · Req 4: Disminuir cantidad -->
                <button
                  class="mini"
                  type="button"
                  @click="disminuir(item.id)"
                >
                  −
                </button>

                <span>x{{ item.cantidad }}</span>

                <!-- B · Req 4: Aumentar cantidad -->
                <button
                  class="mini"
                  type="button"
                  @click="aumentar(item.id)"
                >
                  +
                </button>

                <!-- B · Req 5: Eliminar línea (con .stop para no disparar el clic de la fila) -->
                <button
                  class="mini del"
                  type="button"
                  @click.stop="eliminar(item.id)"
                >
                  ✕
                </button>
              </div>
            </div>
          </div>

          <!-- B · Req 6: Nombre del cliente con v-model -->
          <label for="cliente">Cliente</label>
          <input
            id="cliente"
            type="text"
            v-model="cliente"
            placeholder="Nombre del cliente"
          />

          <!-- ========== EXTRA NIVEL 3 — NOTA PARA LA COCINA (Persona A) ========== -->
          <label for="nota">Nota para la cocina</label>
          <!-- v-model para el texto actual + @keydown.enter para agregar con Enter -->
          <input
            id="nota"
            type="text"
            v-model="notaActual"
            placeholder="Ej: sin cebolla"
            @keydown.enter.prevent="agregarNota"
          />

          <!-- Lista de notas visibles, se llenan al presionar Enter -->
          <div v-if="notas.length">
            <span class="pill" v-for="(n, i) in notas" :key="i">
              {{ n }}
            </span>
          </div>

          <!-- A · Req 2: TOTAL del pedido como valor derivado (computed) -->
          <div class="total">
            TOTAL: ${{ totalConDescuento.toLocaleString('es-CL') }}
          </div>

          <!-- ========== EXTRA NIVEL 3 — DESCUENTO 10% (Persona A) ========== -->
          <!-- .once: el descuento solo se puede aplicar una vez por pedido -->
          <button
            class="btn"
            type="button"
            @click.once="aplicarDescuento"
            :disabled="descuentoAplicado"
            style="margin-top: .5rem; margin-right: .5rem;"
          >
            Aplicar descuento 10%
          </button>

          <!-- Botón principal de cobro -->
          <button type="submit" class="btn">Cobrar y cerrar</button>
        </form>

        <!-- Mensaje de cobro (Persona B) -->
        <p v-if="mensaje" class="pill">{{ mensaje }}</p>
      </div>
    </section>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  name: 'ComandaPos',
  setup() {
    // ===================== DATOS FIJOS DEL MENÚ (NO SE TOCAN) =====================
    // Menú del food truck — cada producto tiene id, emoji, nombre y precio.
    const menu = ref([
      { id: 1, emoji: '🍔', nombre: 'Hamburguesa', precio: 5500 },
      { id: 2, emoji: '🌭', nombre: 'Completo',     precio: 3200 },
      { id: 3, emoji: '🍕', nombre: 'Pizza',        precio: 6800 },
      { id: 4, emoji: '🥤', nombre: 'Bebida',       precio: 1500 },
      { id: 5, emoji: '🍟', nombre: 'Papas',        precio: 2900 },
      { id: 6, emoji: '🌮', nombre: 'Taco',         precio: 4100 },
    ])

    // ===================== ESTADO DEL PEDIDO (A + B) =====================
    // El ticket empieza vacío; cada item tendrá:
    // { id, emoji, nombre, precio, cantidad }
    const pedido = ref([])

    // Nombre del cliente que escribe Dani en la comandera (B · v-model)
    const cliente = ref('')

    // Mensaje que se muestra al cobrar, por ejemplo:
    // "✅ Pedido de Dani cobrado: $12.500" (B)
    const mensaje = ref('')

    // ===================== EXTRA NIVEL 3 — NOTAS DE COCINA (Persona A) =====================
    // Texto actual de la nota que se está escribiendo
    const notaActual = ref('')
    // Lista de notas acumuladas para la cocina
    const notas = ref([])

    // ===================== EXTRA NIVEL 3 — DESCUENTO 10% (Persona A) =====================
    // Flag para saber si ya se aplicó el descuento en este pedido
    const descuentoAplicado = ref(false)

    // ===================== MÉTODOS — PERSONA A (AGREGAR Y TOTAL) =====================

    // A · Req 1: Agregar productos al ticket.
    // - Si el producto ya está en "pedido", se suma 1 a su cantidad.
    // - Si no está, se agrega con cantidad = 1.
    const agregar = (producto) => {
      const existente = pedido.value.find(item => item.id === producto.id)

      if (existente) {
        existente.cantidad += 1
      } else {
        pedido.value.push({
          id: producto.id,
          emoji: producto.emoji,
          nombre: producto.nombre,
          precio: producto.precio,
          cantidad: 1,
        })
      }
    }

    // A · Req 2: TOTAL del pedido como valor derivado (computed).
    // No se guarda a mano; se calcula siempre desde "pedido".
    const total = computed(() => {
      return pedido.value.reduce(
        (suma, item) => suma + item.precio * item.cantidad,
        0
      )
    })

    // Valor derivado que considera el descuento si ya fue aplicado.
    // Si descuentoAplicado es true, totalConDescuento es total * 0.9.
    const totalConDescuento = computed(() => {
      if (!descuentoAplicado.value) {
        return total.value
      }
      return Math.round(total.value * 0.9) // 10% menos, redondeado
    })

    // ===================== MÉTODOS — PERSONA B (EDICIÓN DEL PEDIDO) =====================

    // B · Req 4: Aumentar cantidad de una línea del ticket.
    const aumentar = (id) => {
      const item = pedido.value.find(p => p.id === id)
      if (item) {
        item.cantidad += 1
      }
    }

    // B · Req 4: Disminuir cantidad.
    // Si la cantidad llega a 0, la línea se elimina del pedido (Req 4).
    const disminuir = (id) => {
      const item = pedido.value.find(p => p.id === id)
      if (!item) return

      item.cantidad -= 1

      if (item.cantidad === 0) {
        pedido.value = pedido.value.filter(p => p.id !== id)
      }
    }

    // B · Req 5: Eliminar una línea completa del ticket (botón ✕).
    const eliminar = (id) => {
      pedido.value = pedido.value.filter(p => p.id !== id)
    }

    // ===================== EXTRA NIVEL 3 — MÉTODOS DE NOTA Y DESCUENTO (Persona A) =====================

    // Nota rápida con Enter:
    // Se agrega la notaActual a la lista "notas" y se limpia el input.
    const agregarNota = () => {
      const texto = notaActual.value.trim()
      if (!texto) return
      notas.value.push(texto)
      notaActual.value = ''
    }

    // Descuento por tecla/botón:
    // Marca descuentoAplicado como true. El cálculo real se refleja en totalConDescuento.
    // El .once en el template asegura que este método se ejecute solo una vez por pedido.
    const aplicarDescuento = () => {
      if (descuentoAplicado.value) return
      descuentoAplicado.value = true
    }

    // ===================== MÉTODO — PERSONA B (COBRAR) =====================

    // B · Req 7: Cobrar sin recargar (@submit.prevent).
    // - Si no hay productos, muestra mensaje de error.
    // - Si hay pedido, arma mensaje con nombre de cliente y total (con descuento si aplica).
    // - Vacía el ticket al cobrar y reinicia notas y descuento.
    const cobrar = () => {
      if (pedido.value.length === 0) {
        mensaje.value = 'No hay productos para cobrar'
        return
      }

      mensaje.value = `✅ Pedido de ${cliente.value || 'Cliente sin nombre'} cobrado: $${totalConDescuento.value.toLocaleString('es-CL')}`

      // Reseteamos el estado del pedido y extras
      pedido.value = []
      notas.value = []
      notaActual.value = ''
      descuentoAplicado.value = false
    }

    // Todo lo que se usa en el template se retorna aquí.
    return {
      menu,
      pedido,
      cliente,
      mensaje,
      notaActual,
      notas,
      descuentoAplicado,
      agregar,
      total,
      totalConDescuento,
      aumentar,
      disminuir,
      eliminar,
      agregarNota,
      aplicarDescuento,
      cobrar,
    }
  }
}
</script>

<style scoped>
/* Estilos del componente (ya listos). Las variables viven en .pos y las heredan los hijos. */
.pos {
  --panel: #1e293b;
  --ink: #e2e8f0;
  --muted: #94a3b8;
  --primary: #42b883;
  --primary-2: #35495e;
  --line: #334155;
  --radius: 12px;
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 1.25rem;
  font-family: "Segoe UI", system-ui, -apple-system, sans-serif;
  color: var(--ink);
}

@media (max-width: 760px) {
  .pos {
    grid-template-columns: 1fr;
  }
}

.card {
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: var(--radius);
  padding: 1.5rem 1.75rem;
}

.card > h2 {
  margin-top: 0;
  color: var(--primary);
  border-bottom: 1px solid var(--line);
  padding-bottom: .6rem;
}

.menu-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
  gap: .7rem;
}

.food-card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: .8rem;
  text-align: center;
  cursor: pointer;
  transition: all .15s;
}

.food-card:hover {
  border-color: var(--primary);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, .08);
}

.food-card .emoji {
  font-size: 2rem;
}

.food-card .nom {
  font-weight: 700;
  color: #334155;
  font-size: .9rem;
  margin: .2rem 0;
}

.food-card .pre {
  color: var(--primary-2);
  font-weight: 700;
}

.ticket {
  background: #0f172a;
  color: #e2e8f0;
  border-radius: 12px;
  padding: 1rem 1.1rem;
}

.ticket h4 {
  margin: 0 0 .6rem;
  color: var(--primary);
}

.ticket .item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: .4rem 0;
  border-bottom: 1px dashed #334155;
  font-size: .9rem;
}

.ticket .qty {
  display: inline-flex;
  gap: .3rem;
  align-items: center;
}

.mini {
  background: var(--primary);
  color: #0f172a;
  border: none;
  border-radius: 6px;
  width: 24px;
  height: 24px;
  font-weight: 800;
  cursor: pointer;
}

.mini.del {
  background: #ef4444;
  color: #fff;
}

.total {
  font-size: 1.4rem;
  font-weight: 800;
  color: var(--primary);
  text-align: right;
  margin-top: .6rem;
}

.vacio {
  color: #64748b;
  font-style: italic;
  text-align: center;
  padding: 1rem 0;
}

label {
  font-weight: 600;
  display: block;
  margin: .5rem 0 .25rem;
  color: #cbd5e1;
}

input[type=text] {
  width: 100%;
  padding: .55rem .7rem;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font-size: .95rem;
  font-family: inherit;
  background: #fff;
  color: #1e293b;
  box-sizing: border-box;
}

input:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(66, 184, 131, .25);
}

.btn {
  background: var(--primary);
  color: #0f172a;
  border: none;
  padding: .6rem 1.2rem;
  border-radius: 8px;
  font-weight: 700;
  cursor: pointer;
  font-size: .95rem;
  margin-top: .4rem;  
}

.pill {
  display: inline-block;
  background: #e2e8f0;
  color: #334155;
  padding: .15rem .6rem;
  border-radius: 999px;
  font-size: .8rem;
  margin: .15rem;
  margin-top: .4rem;  
}
</style>