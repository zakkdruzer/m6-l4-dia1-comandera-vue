# 🚚 La Ruta del Sabor — Comandera de Food Truck

Aplicación POS sencilla construida con **Vue 3 + Vite**, pensada para ayudar a Dani a manejar los pedidos de su food truck sin errores y sin papelitos que se pierden.

## 🎯 Objetivo

Simular una **comandera digital** (Punto de Venta) que permita:

- Tocar productos del menú para agregarlos a un ticket.
- Ver y editar el pedido (cantidades, eliminar items).
- Calcular el total automáticamente.
- Registrar el nombre del cliente.
- Cobrar el pedido sin recargar la página.
- Opcional: agregar notas para la cocina y aplicar un descuento del 10%.

---

## 🧱 Tecnologías usadas

- [Vue 3](https://vuejs.org/) (Composition API)
- [Vite](https://vitejs.dev/) como bundler y dev server
- JavaScript (ES Modules)
- HTML + CSS (estilos incluidos en el componente)

---

## 🧩 Estructura del proyecto

- `src/App.vue`  
  Componente raíz que monta la comandera:

  ```vue
  <template>
    <ComandaPos />
  </template>

  <script setup>
  import ComandaPos from './components/ComandaPos.vue'
  </script>
  ```

- `src/components/ComandaPos.vue`  
  Componente principal con:
  - Zona de **MENÚ** (agregar productos).
  - Zona de **TICKET** (gestionar el pedido, notas, descuento y cobro).

---

## 👤 Roles y funcionalidades

### Persona A — Front de venta (agregar y totalizar)

- **Menú interactivo**
  - Lista de productos (`menu`) con `id`, `emoji`, `nombre`, `precio`.
  - Cada tarjeta usa `@click="agregar(p)"` para añadir el producto al pedido.

- **Estado del pedido**
  - `pedido` es un `ref([])` que guarda items con forma:
    ```js
    { id, emoji, nombre, precio, cantidad }
    ```

- **Agregar producto**
  - `agregar(producto)`:
    - Si el producto ya existe en `pedido`, incrementa `cantidad`.
    - Si no existe, lo agrega con `cantidad: 1`.

- **Total del pedido (valor derivado)**
  - `total` es un `computed` que suma `precio * cantidad` de todos los items.
  - La vista usa `TOTAL: ${{ totalConDescuento.toLocaleString('es-CL') }}`.

---

### Persona B — Gestión del pedido (editar y cobrar)

- **Editar cantidades**
  - `aumentar(id)`: sube la cantidad de un item.
  - `disminuir(id)`:
    - Baja la cantidad.
    - Si llega a 0, elimina el item del pedido.

- **Eliminar línea completa**
  - `eliminar(id)` borra el producto del arreglo `pedido`.
  - El botón usa `@click.stop="eliminar(item.id)"` para que el clic no se propague a la fila entera.

- **Nombre del cliente**
  - `cliente` es un `ref('')`.
  - Input con `v-model="cliente"` enlaza el nombre de forma reactiva.

- **Cobrar sin recargar**
  - Formulario con `@submit.prevent="cobrar"`:
    - Si `pedido` está vacío: muestra mensaje “No hay productos para cobrar”.
    - Si hay pedido:
      - Arma mensaje: `✅ Pedido de {cliente} cobrado: $XX`.
      - Vacía el ticket.
      - Reinicia notas y descuento.

---

## 🔴 Extras — Nivel 3

### Nota rápida para la cocina (Persona A)

- **Estado**
  - `notaActual`: texto que se escribe en el input de nota.
  - `notas`: arreglo de notas agregadas.

- **Interacción**
  - Input “Nota para la cocina” con:
    - `v-model="notaActual"`
    - `@keydown.enter.prevent="agregarNota"`
  - Al presionar Enter:
    - Se agrega `notaActual` a `notas`.
    - Se limpia el input.
  - Las notas se muestran como “pills” debajo del input.

### Descuento 10% con `.once` (Persona A)

- **Estado**
  - `descuentoAplicado`: `ref(false)` para saber si ya se aplicó el descuento.

- **Total con descuento**
  - `totalConDescuento` (`computed`):
    - Si `descuentoAplicado` es `false`: devuelve `total`.
    - Si `true`: devuelve `total * 0.9` (10% menos).

- **Interacción**
  - Botón “Aplicar descuento 10%”:
    - `@click.once="aplicarDescuento"` → solo funciona la primera vez por pedido.
    - `:disabled="descuentoAplicado"` → se desactiva cuando ya está aplicado.

---

## ⚙️ Scripts del proyecto

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "predeploy": "npm run build",
    "deploy": "gh-pages -d dist"
  }
}
```

- `dev`: levantar entorno de desarrollo (`npm run dev`).
- `build`: generar versión de producción (`dist`).
- `preview`: previsualizar el build localmente.
- `predeploy` + `deploy`: publicar la carpeta `dist` en la rama `gh-pages` usando `gh-pages`.

---

## 🚀 Deploy en GitHub Pages

1. Configurar `vite.config.js`:

   ```js
   import { defineConfig } from 'vite'
   import vue from '@vitejs/plugin-vue'

   export default defineConfig({
     plugins: [vue()],
     base: '/NOMBRE-DEL-REPO/', // mismo nombre del repo en GitHub
   })
   ```

2. Instalar `gh-pages`:

   ```bash
   npm install gh-pages --save-dev
   ```

3. Ejecutar deploy:

   ```bash
   npm run deploy
   ```

4. En GitHub → Settings → Pages:
   - Source: `Deploy from a branch`.
   - Branch: `gh-pages`, folder `/`.

La app quedará disponible en:

```text
https://TU-USUARIO.github.io/NOMBRE-DEL-REPO/
```

---

## 🧪 Flujo completo esperado

1. Tocar 3 productos distintos → aparecen en el ticket con cantidad 1.
2. Tocar varias veces el mismo → la cantidad aumenta (no se duplica la línea).
3. El total cuadra con la suma real.
4. Usar + y − → las cantidades se modifican y si llega a 0, el producto desaparece.
5. Usar ✕ → se elimina la línea sin afectar otros eventos.
6. Escribir nombre del cliente, nota de cocina, aplicar (opcional) descuento y cobrar:
   - Se muestra mensaje correcto.
   - El ticket se vacía.
   - No se recarga la página.

---

## Puedes ver el resultado en:

https://zakkdruzer.github.io/m6-l4-dia1-comandera-vue/
