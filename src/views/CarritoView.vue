<template>
    <div>
        <!-- Login Form -->
        <div id="loginForm" class="container" v-if="!isLoggedIn">
            <h2>Login</h2>
            <h3>Ingrese su DNI</h3>
            <input type="text" v-model="DNI" class="form-control input-style" placeholder="DNI" required id="DNI">
            <h3>Ingrese el nombre de usuario</h3>
            <input type="text" v-model="username" class="form-control input-style" placeholder="usuario" required
                id="username">
            <h3>Ingrese la clave</h3>
            <input type="password" v-model="password" class="form-control input-style" placeholder="clave" required
                id="password">
            <button class="btn btn-primary btn-style" @click="login">Ingresar/Registrar</button>
            <br>
            <div class="separador"></div>
            <RouterLink to="/">
                <button class="btn btn-secondary btn-style">Volver</button>
            </RouterLink>
        </div>

        <!-- Página Principal -->
        <div id="paginaPrincipal" v-if="isLoggedIn">
            <header id="cabeceralogo" class="bg-light p-3 w-100">
                <div class="titulo">
                    <h1> PEPES SEX SHOP</h1>
                </div>
                <button id="logout" class="btn btn-danger" @click="logout">Cerrar Sesion</button>
            </header>

            <nav id="menuprincipal" class="navbar navbar-expand-lg navbar-light bg-light w-100">
                <div class="container-fluid">
                    <div class="collapse navbar-collapse justify-content-center" id="navbarNav">
                        <ul class="navbar-nav justify-content-center">
                            <RouterLink to="/" class="text-white nav-link">
                                <i class="fas fa-database"></i> Volver
                            </RouterLink>
                        </ul>
                    </div>
                </div>
            </nav>

            <main style="font-family: 'Roboto', sans-serif; color: #f4f4f4; padding: 20px; background-color: #1a1a1a;">
    <section id="elementosCarrito" ref="elementosCarrito" style="display: flex; flex-direction: column; align-items: center; padding: 20px;">
        <!-- Elementos del carrito aquí -->
    </section>
    <aside style="background-color: #333; border-radius: 15px; padding: 20px; box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5); margin-top: 20px;">
        <h2 style="color: #ff004f; text-align: center;">Precio a pagar</h2>
        <p style="font-size: 1.2rem;">Subtotal: $<span style="font-weight: bold;">{{ subtotal.toFixed(2) }}</span></p>
        <p style="font-size: 1.2rem;">IVA (<span style="font-weight: bold;">{{ iva }}</span>%): $<span style="font-weight: bold;">{{ tiva.toFixed(2) }}</span></p>
        <p style="font-size: 1.5rem; color: #ff69b4;">Total: $<span style="font-weight: bold;">{{ total.toFixed(2) }}</span></p>
        <button @click="mostrarDatosFactura" style="padding: 10px 20px; background-color: #ff004f; color: #fff; border: none; border-radius: 25px; font-size: 1rem; cursor: pointer; transition: background-color 0.3s ease, transform 0.3s ease; width: 100%;">
            Pagar
        </button>
    </aside>
</main>


            <footer id="pielogo" class="bg-light w-100">
                <div class="container-fluid">
                    <div class="row">
                        <section class="col-md-4">
                            <h2>&copy; 2025 PEPE'S SEX SHOP. Todos los derechos reservados.</h2>
                        </section>
                    </div>
                </div>
            </footer>
        </div>

        <!-- Modal -->
        <div id="modal" class="modal" v-if="isModalVisible">
            <div class="modal-content">
                <button class="close" @click="isModalVisible = false">X</button>
                <h2>Datos de la factura</h2>
                <label>Ingrese el correo:</label>
                <input type="text" v-model="correo" placeholder="Correo" required id="correo">
                <label>Ingrese la dirección:</label>
                <input type="text" v-model="direccion" placeholder="Dirección" required id="direccion">
                <label>Ingrese el número de teléfono:</label>
                <input type="number" v-model="telefono" placeholder="Teléfono" required id="telefono">
                <button @click="pagar">Pagar en Efectivo</button>
                <div class="separador"></div>
                <button @click="pagarDesdeBanca">Pagar desde la Banca</button>
            </div>
        </div>

    </div>
</template>

<script>
import axios from "axios";
export default {
    data() {
        return {
            DNI: "",
            username: "",
            password: "",
            isLoggedIn: false,
            isModalVisible: false,
            correo: "",
            direccion: "",
            telefono: "",
            subtotal: 0, // Inicializa con 0
            iva: 0,      // Inicializa con 0
            tiva: 0,     // Inicializa con 0
            total: 0,    // Inicializa con 0
        };
    },
    methods: {
        async login() {
            if (this.DNI && this.username && this.password) {
                try {
                    // Realizar la llamada al API
                    const response = await fetch(`http://sexshop.runasp.net/api/Usuario?DNI=${encodeURIComponent(this.DNI)}`);

                    if (!response.ok) {
                        throw new Error("Error al obtener el usuario");
                    }

                    const usuario = await response.json();

                    // Validar el usuario
                    if (usuario && usuario.USU_NOMBRE === this.username) {
                        if (usuario.USU_CONTRASENA === this.password) {
                            // Guardar cookies y actualizar el estado de la sesión
                            this.setCookie("dni", this.DNI);
                            this.setCookie("username", this.username);
                            this.isLoggedIn = true;
                        } else {
                            alert("Contraseña incorrecta.");
                        }
                    } else {
                        alert("Usuario no encontrado. Creando nuevo usuario.");
                        await this.crearUsuario(); // Llama a la función para crear un usuario
                    }
                } catch (error) {
                    console.error("Error al obtener usuario:", error);
                    alert("Error al intentar iniciar sesión.");
                }
            } else {
                alert("Por favor ingrese DNI, usuario y contraseña.");
            }
        },
        logout() {
            this.deleteCookie("username");
            this.deleteCookie("dni");
            this.isLoggedIn = false;
        },
        setCookie(name, value) {
            document.cookie = `${name}=${encodeURIComponent(value)}; path=/`;
        },
        deleteCookie(name) {
            document.cookie = `${name}=; path=/; expires=Thu, 01 Jan 1970 00:00:00 GMT`;
        },
        getCookie(name) {
            const cookieArray = document.cookie.split("; ");
            for (const cookie of cookieArray) {
                const [key, value] = cookie.split("=");
                if (key === name) return decodeURIComponent(value);
            }
            return null;
        },
        checkSession() {
            const dni = this.getCookie("dni");
            const username = this.getCookie("username");

            if (dni && username) {
                this.DNI = dni;
                this.username = username;
                this.isLoggedIn = true; // Actualizar estado
            } else {
                this.isLoggedIn = false; // Asegurar estado inicial
            }
        },

        async verCarrito() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const contenedor = this.$refs.elementosCarrito;
            if (!contenedor) {
                return;
            }

            contenedor.innerHTML = ''; // Limpiar el contenedor

            if (carrito.length === 0) {
                contenedor.innerHTML = '<p>No hay productos en el carrito.</p>';
                return;
            }

            const tituloCarrito = document.createElement('h2');
            const botonBorrar = document.createElement('button');
            botonBorrar.textContent = 'Limpiar Carrito';
            botonBorrar.onclick = () => this.limpiarCarrito();
            tituloCarrito.textContent = 'Tu Carrito';
            contenedor.appendChild(tituloCarrito);
            contenedor.appendChild(botonBorrar);

            try {
                const response = await fetch('http://sexshop.runasp.net/api/Producto');
                if (!response.ok) throw new Error('Error al obtener productos desde la API');
                const productos = await response.json();

                carrito.forEach(item => {
                    const productoBaseDatos = productos.find(p => p.PRD_ID == item.id);
                    if (!productoBaseDatos) return; // Si no existe, saltar

                    const producto = document.createElement('div');
                    producto.classList.add('producto');

                    producto.innerHTML = `
<div class="imagen-producto" style="background-image: url(${productoBaseDatos.PRD_IMAGEN}); background-size: cover; background-position: center; width: 100%; height: 200px; border-radius: 15px; box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);"></div>

<div class="info-producto" style="background-color: #2a2a2a; padding: 15px; border-radius: 15px; margin-top: 10px; text-align: center;">
    <h3 style="color: #ff69b4; font-size: 1.8rem; margin: 0;">${productoBaseDatos.PRD_NOMBRE}</h3>
    <p style="font-size: 1.2rem; color: #ff004f; margin: 10px 0;">Precio: $${productoBaseDatos.PRD_PRECIO}</p>
</div>

<div class="controles" data-id="${item.nombre}" style="display: flex; justify-content: space-between; align-items: center; margin-top: 20px; background-color: #333; padding: 10px; border-radius: 15px;">
    <button class="disminuir" style="padding: 8px 12px; background-color: #ff004f; color: #fff; border: none; border-radius: 25px; cursor: pointer; transition: background-color 0.3s ease;">
        -
    </button>
    <input type="number" value="${item.cantidad}" readonly max="${productoBaseDatos.PRD_STOCK}" style="width: 50px; text-align: center; background-color: #333; border: 1px solid #ff004f; color: #fff; font-size: 1.2rem; border-radius: 10px;">
    <button class="aumentar" style="padding: 8px 12px; background-color: #ff004f; color: #fff; border: none; border-radius: 25px; cursor: pointer; transition: background-color 0.3s ease;">
        +
    </button>
    <button class="eliminar" style="padding: 8px 12px; background-color: #f05656; color: #fff; border: none; border-radius: 25px; cursor: pointer; transition: background-color 0.3s ease;">
        Eliminar
    </button>
</div>

            `;
                    const eliminarButton = producto.querySelector('.eliminar');
                    eliminarButton.addEventListener('click', () => {
                        this.quitarDelCarrito(item.nombre);
                    });

                    const aumentarButton = producto.querySelector('.aumentar');
                    aumentarButton.addEventListener('click', () => {
                        this.aumentarProducto(item.nombre, productoBaseDatos.PRD_STOCK);
                    });

                    const disminuirButton = producto.querySelector('.disminuir');
                    disminuirButton.addEventListener('click', () => {
                        this.disminuirProducto(item.nombre);
                    });

                    contenedor.appendChild(producto);
                });
            } catch (error) {
                console.error('Error al procesar el carrito:', error);
                alert('No se pudieron cargar los productos.');
            }
        },

        quitarDelCarrito(nombreProducto) {
            let carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);
            if (index !== -1) {
                carrito.splice(index, 1);
                localStorage.setItem('carrito', JSON.stringify(carrito));
                this.verCarrito();
                this.calcularTotal();
            } else {
                console.log('El producto no se encontró en el carrito.');
            }
        },

        aumentarProducto(nombreProducto, maxStock) {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);

            if (index !== -1) {
                if (carrito[index].cantidad < maxStock) {
                    carrito[index].cantidad++;
                    localStorage.setItem('carrito', JSON.stringify(carrito));

                    // Actualiza el valor del input en el HTML
                    const inputCantidad = document.querySelector(`div[data-id='${nombreProducto}'] input[type='number']`);
                    if (inputCantidad) {
                        inputCantidad.value = carrito[index].cantidad;
                    }

                    this.calcularTotal();
                } else {
                    alert('Has alcanzado el límite de stock disponible para este producto.');
                }
            }
        },

        disminuirProducto(nombreProducto) {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const index = carrito.findIndex(item => item.nombre === nombreProducto);
            if (index !== -1) {
                carrito[index].cantidad--;
                if (carrito[index].cantidad > 0) {
                    localStorage.setItem('carrito', JSON.stringify(carrito));

                    // Actualiza el valor del input en el HTML
                    const inputCantidad = document.querySelector(`div[data-id='${nombreProducto}'] input[type='number']`);
                    if (inputCantidad) {
                        inputCantidad.value = carrito[index].cantidad;
                    }

                    this.calcularTotal();
                } else {
                    this.quitarDelCarrito(nombreProducto);
                }
            }
        },

        limpiarCarrito() {
            localStorage.removeItem('carrito');
            this.verCarrito();
            this.calcularTotal();
        },

        async calcularTotal() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const id = 1; // El ID que se utiliza según tu código en el controlador

            try {
                const response = await fetch(`http://sexshop.runasp.net/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }

                const datosGenerales = await response.json();
                if (datosGenerales) {
                    const iva = datosGenerales.IVA;
                    const subtotal = carrito.reduce((sum, item) => sum + item.precio * item.cantidad, 0);
                    const tiva = subtotal * (iva / 100);
                    const total = subtotal + tiva;

                    // Actualiza las propiedades del estado
                    this.subtotal = subtotal;
                    this.iva = iva;
                    this.tiva = tiva;
                    this.total = total;
                } else {
                    console.error('No se encontraron datos generales.');
                }
            } catch (error) {
                console.error('Error:', error);
            }
        },

        mostrarDatosFactura() {
            const username = this.getCookie("username");

            if (username) {
                const carrito = JSON.parse(localStorage.getItem("carrito")) || [];
                if (carrito.length > 0) {
                    this.isModalVisible = true;
                } else {
                    alert("No hay objetos en el carrito.");
                }
            } else {
                alert("Por favor inicie sesión para continuar con el pago.");
            }
        },

        async pagarDesdeBanca() {
            const username = this.getCookie("username");
            const dni = this.getCookie("dni");
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            if (!username) {
                alert('Por favor inicie sesión para continuar con el pago.');
                return;
            }

            if (carrito.length === 0) {
                alert('No hay objetos en el carrito');
                return;
            }

            try {
                // Calcular el total de la factura
                const iva = this.iva;
                const subtotal = this.subtotal;
                const tiva = this.tiva;
                const totalFac = this.total;
                const id = 1;

                const response = await fetch(`http://sexshop.runasp.net/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();

                // Verificar que la propiedad NUM_FAC existe
                if (!datosGenerales.NUM_FAC) {
                    throw new Error('Número de factura no disponible');
                }

                // Obtener datos adicionales para la factura
                const correo = document.getElementById('correo').value;
                const direcFac = document.getElementById('direccion').value;
                const telfFac = document.getElementById('telefono').value;
                const fechFac = new Date().toISOString();
                const facActual = "FAC" + datosGenerales.NUM_FAC;

                // Crear objeto de factura que coincida con el modelo FACTURA en la API
                const factura = {
                    FAC_NUMERO: facActual,
                    USU_DNI: dni,
                    USU_NOMBRE: username,
                    USU_CORREO: correo,
                    FAC_DIRECCION: direcFac,
                    FAC_TELEFONO: telfFac,
                    FAC_FECHA: fechFac,
                    FAC_TOTAL: totalFac,
                    FAC_ESTADO: "Pendiente"
                };

                // Guardar la factura en el servidor llamando al controlador Factura/Create
                const saveResponse = await axios.post(`http://sexshop.runasp.net/api/Factura`, factura);

                // No es necesario verificar saveResponse.status ya que Axios lanzará un error si hay un problema
                alert('Factura guardada correctamente.');

                // Llamadas a funciones adicionales para completar el proceso de pago
                this.agregarDetalleFactura(); // Si es necesario
                this.aumentarNumeroFactura(); // Si es necesario
                this.limpiarCarrito();

                // Limpiar campos del formulario y cerrar modal
                document.getElementById('modal').style.display = 'none';
                document.getElementById('direccion').value = '';
                document.getElementById('telefono').value = '';
                document.getElementById('correo').value = '';

            } catch (error) {
                console.error('Error al procesar el pago:', error);
                alert('Hubo un error al procesar el pago.');
            }
        },

        async pagar() {
            const username = this.getCookie("username");
            const dni = this.getCookie("dni");
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            if (!username) {
                alert('Por favor inicie sesión para continuar con el pago.');
                return;
            }

            if (carrito.length === 0) {
                alert('No hay objetos en el carrito');
                return;
            }

            try {
                // Calcular el total de la factura
                const iva = this.iva;
                const subtotal = this.subtotal;
                const tiva = this.tiva;
                const totalFac = this.total;
                const id = 1;

                const response = await fetch(`http://sexshop.runasp.net/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();

                // Verificar que la propiedad NUM_FAC existe
                if (!datosGenerales.NUM_FAC) {
                    throw new Error('Número de factura no disponible');
                }

                // Obtener datos adicionales para la factura
                const correo = document.getElementById('correo').value;
                const direcFac = document.getElementById('direccion').value;
                const telfFac = document.getElementById('telefono').value;
                const fechFac = new Date().toISOString();
                const facActual = "FAC" + datosGenerales.NUM_FAC;

                // Crear objeto de factura que coincida con el modelo FACTURA en la API
                const factura = {
                    FAC_NUMERO: facActual,
                    USU_DNI: dni,
                    USU_NOMBRE: username,
                    USU_CORREO: correo,
                    FAC_DIRECCION: direcFac,
                    FAC_TELEFONO: telfFac,
                    FAC_FECHA: fechFac,
                    FAC_TOTAL: totalFac,
                    FAC_ESTADO: "Pagado"
                };

                // Guardar la factura en el servidor llamando al controlador Factura/Create
                const saveResponse = await axios.post(`http://sexshop.runasp.net/api/Factura`, factura);

                // No es necesario verificar saveResponse.status ya que Axios lanzará un error si hay un problema
                alert('Factura guardada correctamente.');

                // Llamadas a funciones adicionales para completar el proceso de pago
                this.agregarDetalleFactura();
                this.debitarProductos();
                this.aumentarNumeroFactura(); // Si es necesario
                this.limpiarCarrito();

                // Limpiar campos del formulario y cerrar modal
                document.getElementById('modal').style.display = 'none';
                document.getElementById('direccion').value = '';
                document.getElementById('telefono').value = '';
                document.getElementById('correo').value = '';

            } catch (error) {
                console.error('Error al procesar el pago:', error);
                alert('Hubo un error al procesar el pago.');
            }
        },

        async agregarDetalleFactura() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];

            try {
                // Obtener datos generales como IVA
                const id = 1;
                const response = await fetch(`http://sexshop.runasp.net/api/DatosGenerales/${id}`);
                if (!response.ok) {
                    throw new Error('Error al obtener los datos generales');
                }
                const datosGenerales = await response.json();
                const iva = datosGenerales.IVA;

                // Generar el número de factura actual (ej. "FAC10")
                const facNumero = "FAC" + datosGenerales.NUM_FAC;

                // Guardar cada detalle de producto en la API
                for (const producto of carrito) {
                    try {
                        const subtotalPrd = (producto.precio * producto.cantidad) * (1 + (iva / 100));

                        const detalleFactura = {
                            FAC_NUMERO: facNumero,
                            PRD_ID: producto.id,
                            PRD_CANTIDAD: producto.cantidad,
                            PRD_SUBTOTAL: subtotalPrd
                        };

                        const detalleResponse = await axios.post(`http://sexshop.runasp.net/api/Detalle_Factura`, detalleFactura);

                        // Manejar la respuesta si es necesario
                        console.log('Detalle guardado:', detalleResponse.data);

                    } catch (error) {
                        console.error('Error al guardar detalle para el producto:', producto, error);
                        continue; // Continúa con el siguiente producto en el carrito
                    }
                }

                alert('Detalles de factura guardados correctamente.');
            } catch (error) {
                console.error('Error al guardar los detalles de factura:', error);
                alert('Hubo un error general al guardar los detalles de factura.');
            }
        },

        async debitarProductos() {
            const carrito = JSON.parse(localStorage.getItem('carrito')) || [];
            const errores = []; // Array para almacenar errores

            for (const producto of carrito) {
                try {
                    const response = await axios.get(`http://sexshop.runasp.net/api/Producto?PrdNombre=${producto.id}`);
                    const productoADebitar = response.data;

                    // Validar que el producto exista
                    if (!productoADebitar || typeof productoADebitar.PRD_STOCK === 'undefined') {
                        throw new Error(`Producto con ID ${producto.PRD_ID} no encontrado o sin stock disponible.`);
                    }

                    productoADebitar.PRD_STOCK = productoADebitar.PRD_STOCK - producto.cantidad;
                    if (productoADebitar.PRD_STOCK < 0) {
                        throw new Error(`Stock insuficiente para el producto ${productoADebitar.PRD_NOMBRE}.`);
                    }


                    // Enviar la actualización al servidor
                    const debitarResponse = await axios.put(`http://sexshop.runasp.net/api/Producto/${producto.PRD_ID}`, productoADebitar);

                } catch (error) {
                    // Manejar errores en la solicitud
                    const errorMessage = error.response?.data?.message || 'No se pudo debitar el producto';
                    console.error(`Error al procesar el producto ${producto.nombre}:`, errorMessage);
                    errores.push(`Error con el producto ID ${producto.PRD_ID}: ${errorMessage}`);
                }
            }

            // Muestra los errores al final, si existen
            if (errores.length > 0) {
                alert(`Se produjeron los siguientes errores:\n- ${errores.join('\n- ')}`);
            }
        },

        async aumentarNumeroFactura() {
            try {
                const id = 1;

                // Obtener los datos generales
                const response = await axios.get(`http://sexshop.runasp.net/api/DatosGenerales/${id}`);
                const datos = response.data;

                // Incrementar el número de factura
                datos.NUM_FAC++;

                // Enviar la actualización al servidor
                const CambiarResponse = await axios.put(`http://sexshop.runasp.net/api/DatosGenerales/${id}`, datos);

                console.log("Número de factura actualizado correctamente:", CambiarResponse.data);
            } catch (error) {
                const errorMessage = error.response?.data?.message || 'No se pudo actualizar el número de factura.';
                console.error("Error al actualizar el número de factura:", errorMessage);
                alert(`Error: ${errorMessage}`);
            }
        },

        async crearUsuario() {
            try {
                const nuevoDNI = document.getElementById("DNI").value;
                const nuevoUsuario = document.getElementById("username").value;
                const nuevaContrasena = document.getElementById("password").value;

                // Enviar la solicitud POST con Axios
                const response = await axios.post('http://sexshop.runasp.net/api/Usuario', {
                    USU_DNI: nuevoDNI,
                    USU_NOMBRE: nuevoUsuario,
                    USU_CONTRASENA: nuevaContrasena
                });

                // Verificar la respuesta
                if (response.status === 201 || response.status === 200) {
                    alert('Usuario creado exitosamente.');
                    console.log('Usuario creado:', response.data);
                } else {
                    console.error('Error al ingresar el nuevo usuario:', response);
                    alert('Hubo un error al ingresar el nuevo usuario.');
                }
            } catch (error) {
                // Manejar errores
                const errorMessage = error.response?.data?.message || 'Error desconocido.';
                console.error('Error al ingresar el nuevo usuario:', errorMessage);
                alert(`Hubo un error al ingresar el nuevo usuario: ${errorMessage}`);
            }
        }


    },
    mounted() {
        this.$nextTick(() => {
            this.checkSession();
            if (this.isLoggedIn) {
                setTimeout(() => {
                    this.verCarrito(); // Asegurar que el DOM esté listo
                }, 100); // Ajusta el tiempo si es necesario
            }
            this.calcularTotal();
        });
    }

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

#loginForm {
  max-width: 400px;
  margin: 40px auto;
  background-color: #2a2a2a;
  border-radius: 15px;
  padding: 20px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);
}

#loginForm h2, #loginForm h3 {
  color: #ff004f; /* Rojo */
  text-align: center;
}

.input-style {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ff004f;
  border-radius: 10px;
  background-color: #333;
  color: #fff;
  font-size: 1.1rem;
}

.input-style:focus {
  outline: none;
  border-color: #ff69b4;
}

.btn-style {
  width: 100%;
  padding: 10px;
  background-color: #ff004f;
  color: #fff;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.btn-style:hover {
  background-color: #ff69b4;
  transform: scale(1.1);
}

/* Página Principal */
#paginaPrincipal {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
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

#logout {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #ff004f;
  color: #fff;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

#logout:hover {
  background-color: #ff69b4;
  transform: scale(1.1);
}

nav#menuprincipal {
  background-color: #121212;
}

nav#menuprincipal .nav-link {
  color: #ff69b4;
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

aside {
  margin-top: 20px;
  padding: 20px;
  background-color: #333;
  border-radius: 15px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);
}

aside h2 {
  color: #ff004f;
}

aside p {
  color: #f4f4f4;
}

footer#pielogo {
  background-color: #121212;
  color: #f4f4f4;
}

footer#pielogo h2 {
  font-size: 1.5rem;
}

#modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #2a2a2a;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);
  max-width: 400px;
  width: 100%;
  color: #f4f4f4;
}

button.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 1.5rem;
  color: #ff004f;
  background: none;
  border: none;
  cursor: pointer;
}

button.close:hover {
  color: #ff69b4;
}

.modal-content h2 {
  text-align: center;
  color: #ff004f;
}

.modal-content input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ff004f;
  border-radius: 10px;
  background-color: #333;
  color: #fff;
  font-size: 1.1rem;
}

.modal-content button {
  width: 100%;
  padding: 10px;
  background-color: #ff004f;
  color: #fff;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.modal-content button:hover {
  background-color: #ff69b4;
  transform: scale(1.1);
}

.separador {
    width: 100%; /* Ocupa todo el ancho */
    height: 2px; /* Espesor de la línea */
    background-color: #e0e0e0; /* Color de la línea */
    margin: 20px 0; /* Espaciado superior e inferior */
  }
</style>