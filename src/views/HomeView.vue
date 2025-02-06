<template>
  <header id="cabeceralogo" class="bg-dark p-3 w-100">
    <h1>PEPE'S SEX SHOP</h1>
  </header>

  <div class="advertisement-container">
  <img src="https://s7.gifyu.com/images/SXehw.gif" alt="Publicidad" class="advertisement-gif" />
</div>

<div class="separador"></div>

      <!-- Tarjetas con preguntas y respuestas -->
      <div class="cards-container">
      <div class="card">
        <h3>¿Es seguro comprar en un sex shop online?</h3>
        <p>¡Sí! En PEPE'S SEX SHOP tomamos todas las medidas necesarias para garantizar la seguridad y privacidad de nuestros clientes. Usamos conexiones encriptadas y procesadores de pagos certificados.</p>
      </div>

      <div class="card">
        <h3>¿Cómo se protege mi privacidad al realizar un pedido?</h3>
        <p>Tu privacidad es nuestra prioridad. Los detalles de tu compra se manejan de forma confidencial y no compartimos tu información con terceros sin tu consentimiento.</p>
      </div>

      <div class="card">
        <h3>¿Puedo confiar en la calidad de los productos?</h3>
        <p>Todos nuestros productos son de alta calidad y cumplen con los estándares más rigurosos de seguridad y bienestar. Si tienes alguna duda, nuestro equipo de soporte está disponible para ayudarte.</p>
      </div>
    </div>

  <nav id="menuprincipal" class="navbar navbar-expand-lg navbar-dark bg-dark w-100">
    <ul class="navbar-nav justify-content-center">
      <li>
        <RouterLink to="/CarritoView" class="text-white nav-link">
          <i class="fas fa-shopping-cart"></i> Carrito
        </RouterLink>
      </li>
    </ul>
  </nav>

  <main id="productos" class="productos-lista">

    <div
      v-for="producto in productos"
      :key="producto.PRD_ID"
      :id="'producto-' + producto.PRD_NOMBRE"
      class="producto"
    >
      <div class="foto-container">
        <div
          id="foto"
          :style="{ backgroundImage: `url(${producto.PRD_IMAGEN})` }"
        ></div>
      </div>
      <div class="producto-info">
        <h2>{{ producto.PRD_NOMBRE }}</h2>
        <h3>Precio: {{ producto.PRD_PRECIO }}$</h3>
        <div class="producto-actions">
          <input
            v-model.number="producto.cantidad"
            type="range"
            :min="0"
            :max="producto.PRD_STOCK"
            :step="1"
            class="slider"
          />
          <span>{{ producto.cantidad }}</span> <!-- Muestra la cantidad seleccionada -->
          <button @click="agregarAlCarrito(producto)">
            <i class="fas fa-heart"></i> Añadir
          </button>
        </div>
      </div>
    </div>
  </main>

  <footer id="pielogo" class="bg-dark text-center p-3">
    <h2>&copy; 2025 PEPE'S SEX SHOP. Todos los derechos reservados.</h2>
  </footer>
</template>

<script>
export default {
  data() {
    return {
      productos: [],
    };
  },
  mounted() {
    this.obtenerProductos();
  },
  methods: {
    async obtenerProductos() {
      try {
        const response = await fetch('http://sexshop.runasp.net/api/Producto');
        if (!response.ok) {
          throw new Error('Error en la respuesta de la API');
        }
        const data = await response.json();
        this.productos = data.map(producto => ({
          ...producto,
          cantidad: 0, // Añadimos la propiedad cantidad
        }));
      } catch (error) {
        console.error('Error al obtener productos:', error);
      }
    },
    agregarAlCarrito(producto) {
      const cantidad = producto.cantidad;
      const maximo = producto.PRD_STOCK;
      const precio = producto.PRD_PRECIO;
      const id = producto.PRD_ID;

      if (cantidad > 0 && cantidad <= maximo) {
        let carrito = JSON.parse(localStorage.getItem('carrito')) || [];
        let item = carrito.find(p => p.id === id);
        if (item) {
          item.cantidad += cantidad;
        } else {
          carrito.push({
            nombre: producto.PRD_NOMBRE,
            cantidad,
            precio,
            id,
          });
        }
        localStorage.setItem('carrito', JSON.stringify(carrito));
        alert('Producto agregado al carrito');
        producto.cantidad = 0; // Limpiar el campo de cantidad después de agregar
      } else {
        alert('Cantidad no válida');
      }
    },
  },
};
</script>

<style>
/* Colores y fuentes sensuales */
body {
  font-family: 'Roboto', sans-serif;
  margin: 0;
  padding: 0;
  background-color: #1a1a1a; /* Fondo negro */
  color: #f4f4f4; /* Texto claro */
}

header#cabeceralogo {
  background: linear-gradient(135deg, #ff004f, #e6003a);
  color: #ffffff;
  text-align: center;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
}

header#cabeceralogo h1 {
  margin: 0;
  font-size: 3rem;
  font-weight: bold;
  text-transform: uppercase;
  text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.6);
}

nav#menuprincipal {
  background-color: #121212;
}

nav#menuprincipal .nav-link {
  color: #ff69b4; /* Rosa brillante */
  font-weight: bold;
  font-size: 1.2rem;
  padding: 8px 20px;
  border-radius: 25px;
  transition: background-color 0.3s, transform 0.2s;
}

nav#menuprincipal .nav-link:hover {
  background-color: #ff004f;
  transform: scale(1.1);
}

#productos {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.producto {
  width: 90%;
  max-width: 600px;
  margin: 20px 0;
  background-color: #2a2a2a;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  text-align: center;
  padding: 15px;
  transition: transform 0.3s ease;
}

.producto:hover {
  transform: scale(1.03);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.7);
}

.foto-container {
  height: 300px;
  background-color: #333;
  margin-bottom: 15px;
}

#foto {
  width: 100%;
  height: 100%;
  background-size: contain;
  background-position: center;
}

.producto-info h2 {
  font-size: 1.8rem;
  color: #ff69b4;
  margin: 10px 0;
  text-transform: uppercase;
}

.producto-info h3 {
  font-size: 1.5rem;
  color: #ff004f;
  margin: 10px 0;
}

.producto-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.producto-actions input[type="range"] {
  width: 100%;
  max-width: 200px;
  margin: 10px 0;
  -webkit-appearance: none;
  background-color: #ff69b4;
  border-radius: 10px;
  height: 10px;
  transition: background-color 0.3s ease;
}

.producto-actions input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background-color: #ff004f;
  border-radius: 50%;
  cursor: pointer;
}

.producto-actions button {
  padding: 10px 20px;
  font-size: 1rem;
  color: #fff;
  background-color: #ff004f;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.producto-actions button:hover {
  background-color: #ff69b4;
  transform: scale(1.1);
}

footer#pielogo {
  background-color: #121212;
  color: #f4f4f4;
}

.cards-container {
    display: flex;
    justify-content: space-between;
    gap: 20px;
  }

  .card {
    background-color: #f8b0b0; /* Color rosado suave */
    border-radius: 10px;
    padding: 20px;
    width: 30%;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: background-color 0.3s ease;
  }

  .card:hover {
    background-color: #f05656; /* Color rojo al pasar el mouse */
  }

  .card h3 {
    color: #7a1d1d; /* Rojo oscuro para los títulos */
  }

  .card p {
    color: #4d4d4d; /* Color gris oscuro para el texto */
  }
  .advertisement-container {
    position: relative;
    width: 100%;
    height: 100vh; /* Ocupa toda la altura de la ventana */
    overflow: hidden;
  }

  .advertisement-gif {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover; /* Asegura que el gif cubra toda el área */
    z-index: -1; /* Asegura que el gif esté de fondo */
  }

  .separador {
    width: 100%; /* Ocupa todo el ancho */
    height: 2px; /* Espesor de la línea */
    background-color: #e0e0e0; /* Color de la línea */
    margin: 20px 0; /* Espaciado superior e inferior */
  }
</style>