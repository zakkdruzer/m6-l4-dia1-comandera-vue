<script>
import { ref, computed } from 'vue'

export default {
  name: 'ComandaPos',
  setup() {
    // ---------- DATOS (ya listos, NO los cambies) ----------
    const menu = ref([
      { id: 1, emoji: '🍔', nombre: 'Hamburguesa', precio: 5500 },
      { id: 2, emoji: '🌭', nombre: 'Completo', precio: 3200 },
      { id: 3, emoji: '🍕', nombre: 'Pizza', precio: 6800 },
      { id: 4, emoji: '🥤', nombre: 'Bebida', precio: 1500 },
      { id: 5, emoji: '🍟', nombre: 'Papas', precio: 2900 },
      { id: 6, emoji: '🌮', nombre: 'Taco', precio: 4100 },
    ])
    
    const pedido = ref([])

  const cliente = ref('')
  const mensaje = ref('')

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

  const total = computed(() => {
    return pedido.value.reduce(
      (suma, item) => suma + item.precio * item.cantidad,
      0
    )
  })

  return {
    menu,
    pedido,
    cliente,
    mensaje,
    agregar,
    total,
  }
}
}
</script>

<template>
  <div class="pos">

    <!-- ================= ZONA MENÚ (EJEMPLO YA HECHO) ================= -->
    <section class="card">
      <h2>🍔 Menú — toca para agregar</h2>

      <div class="food-card" v-for="p in menu" :key="p.id" @click="agregar(p)">
        <div class="emoji">{{ p.emoji }}</div>
        <div class="nom">{{ p.nombre }}</div>
        <div class="pre">${{ p.precio.toLocaleString('es-CL') }}</div>
      </div>
    </section>

    <!-- ================= ZONA TICKET (LA CONSTRUYEN USTEDES) ================= -->
    <section class="card">
      <h2>🧾 Ticket del pedido</h2>

      <!--
        🔨 CONSTRUYAN USTEDES TODO ESTE PANEL. Guíense por el menú de arriba (v-for)
           y por los Requisitos 1 a 7 de las instrucciones. Debe tener:

           [B · Req 6] Un input para el nombre del cliente     -> v-model="cliente"
           [A · Req 3] Un mensaje de "Aún no hay productos"    -> v-if cuando el pedido esté vacío
           [A · Req 1] La lista del pedido                     -> v-for sobre "pedido"
                       mostrando emoji, nombre y cantidad de cada item
           [B · Req 4] Botones  −  y  +  en cada línea         -> @click a disminuir(id) / aumentar(id)
           [B · Req 5] Un botón  ✕  para eliminar la línea     -> @click.stop a eliminar(id)  (¡ojo el .stop!)
           [A · Req 2] El TOTAL del pedido                     -> {{ total }}  (un computed)
           [B · Req 7] Un <form> con botón "Cobrar y cerrar"   -> @submit.prevent="cobrar"

        Pueden usar las clases de CSS ya listas (ver la sección <style> más abajo):
        .ticket, .item, .qty, .mini, .mini.del, .total, .vacio, .btn
      -->
      <div class="ticket">
        <h4>Pedido</h4>

        <div v-if="pedido.length === 0" class="vacio">
          Aún no hay productos
        </div>

        <div v-else>
          <div class="item" v-for="item in pedido" :key="item.id">
            <div>
              <span>{{ item.emoji }}</span>
              <span> {{ item.nombre }} </span>
            </div>

            <div class="qty">
              <!-- Botones +, -, ✕ los añadiremos en el rol B -->
              <span>x{{ item.cantidad }}</span>
            </div>
          </div>
        </div>

        <label for="cliente">Cliente</label>
        <input id="cliente" type="text" v-model="cliente" placeholder="Nombre del cliente" />

        <div class="total">
          TOTAL: ${{ total.toLocaleString('es-CL') }}
        </div>

      </div>
    </section>

  </div>
</template>

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

.card>h2 {
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
}

.pill {
  display: inline-block;
  background: #e2e8f0;
  color: #334155;
  padding: .15rem .6rem;
  border-radius: 999px;
  font-size: .8rem;
  margin: .15rem;
}
</style>