<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Lista de Productos</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="nuevoProducto = true">
            <ion-icon name="add"></ion-icon> Agregar Producto
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <ion-row class="d-flex align-center justify-center animated-row fade-in">
        <ion-col cols="12" md="6" class="animated-column fade-in">
          <ion-item>
            <ion-label position="floating">Buscar por ID</ion-label>
            <ion-input v-model="searchById" @input="buscarProductoId"></ion-input>
            <ion-icon slot="start" :icon="searchIcon"></ion-icon>
          </ion-item>
        </ion-col>
        <ion-col cols="12" md="6" class="animated-column fade-in">
          <ion-item>
            <ion-label position="floating">Buscar por Categoría</ion-label>
            <ion-input v-model="searchByCategory" @input="buscarProductoCategoria"></ion-input>
            <ion-icon slot="start" :icon="searchIcon"></ion-icon>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row class="d-flex align-center justify-center">
        <ion-col v-for="producto in products" :key="producto.id" size-md="3">
          <ion-card :id="'card-' + producto.id" class="animated-card fade-in mx-auto align-center hover-elevate"
            max-width="250" style="border: 5px solid #b46474" elevation="10">
            <ion-img :src="producto.imagenArchivo" height="200px" />
            <ion-card-header class="font-weight-bold typewriter-effect">
              <ion-card-title>{{ producto.nombre }}</ion-card-title>
            </ion-card-header>
            <hr />
            <ion-card-content>
              <ion-card-subtitle>
                <ion-card-text class="font-weight-bold">ID: {{ producto.id }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold">Categoría: {{ producto.categoria }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold">Resumen: {{ producto.resumen }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold">Descripción: {{ producto.descripcion }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold">Precio: ${{ producto.precio.toFixed(2) }}</ion-card-text>
              </ion-card-subtitle>
            </ion-card-content>
            <ion-row class="mx-auto text-center my-4">
              <ion-col>
                <!-- Botón de eliminar -->
                <ion-button block color="danger" fill="outline" @click="confirmarEliminacion(producto.id)">
                  <ion-icon slot="start" name="trash-outline"></ion-icon>Eliminar
                </ion-button>
              </ion-col>
              <ion-col>
                <!-- Botón de actualizar -->
                <ion-button block color="warning" fill="outline" @click="abrirModal(producto)">
                  <ion-icon slot="start" name="create-outline"></ion-icon>Actualizar
                </ion-button>
              </ion-col>
            </ion-row>
          </ion-card>
        </ion-col>
      </ion-row>
      <!-- Componente del formulario de guardar producto -->
      <GuardarProductoModal v-bind:nuevoProducto="nuevoProducto" formTitle="Nuevo Producto" @dismiss="resetModal"
        @actualizarProductos="obtenerProductos" />

      <EditarProductoModal :productoSeleccionado="productoSeleccionado" :editarProductoModal="mostrarModalEdicion"
        @dismiss="resetModal" :is-open="mostrarModalEdicion" />

      <ion-toast :is-open="showToast" :message="toastMessage" :color="toastColor" position="bottom"
        :duration="3000"></ion-toast>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonRow, IonCol, IonCard, IonImg, IonCardHeader, IonCardTitle, IonCardContent, IonButton, IonIcon, IonFab, IonFabButton, IonItem, IonLabel, IonInput, IonAlert } from '@ionic/vue';
import { defineComponent } from 'vue';
import swal from 'sweetalert';
import GuardarProductoModal from "@/components/NuevoProducto.vue";
import EditarProductoModal from "@/components/EditarProducto.vue";
export default defineComponent({
  components: {
    IonPage,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonRow,
    IonCol,
    IonCard,
    IonImg,
    IonCardHeader,
    IonCardTitle,
    IonCardContent,
    IonButton,
    IonIcon,
    IonFab,
    IonFabButton,
    IonItem,
    IonLabel,
    IonInput,
    IonAlert,
    GuardarProductoModal,
    EditarProductoModal

  },
  data() {
    return {
      products: [],
      imagen: 'https://img.freepik.com/vector-premium/dibujos-animados-productos-supermercado_24640-55628.jpg',
      nuevoProducto: false,
      formTitle: '',
      nombre: '',
      categoria: '',
      resumen: '',
      descripcion: '',
      imagenArchivo: '',
      precio: '',
      showToast: false,
      toastMessage: '',
      toastColor: '',
      searchByCategory: '',
      searchById: '',
      productoSeleccionado: {
        id: '',
        nombre: '',
        categoria: '',
        resumen: '',
        descripcion: '',
        imagenArchivo: '',
        precio: ''
      }, // Asegúrate de tener el objeto del producto seleccionado
      mostrarModalEdicion: false
    };
  },
  created() {
    this.mostrarProductos();
  },
  methods: {
    resetModal() {
      this.nuevoProducto = false;
      this.mostrarModalEdicion = false;
      this.formTitle = '';
      this.nombre = '';
      this.categoria = '';
      this.resumen = '';
      this.descripcion = '';
      this.imagenArchivo = '';
      this.precio = '';
      this.mostrarModalEdicion = false;
    },
    async mostrarProductos() {
      try {
        const response = await fetch('http://192.168.0.2:550/api/v1/Catalog');
        if (!response.ok) {
          swal("Error", "No Se Puede Obtener los Productos", "error")
        }
        const data = await response.json();
        this.products = data;
        this.$refs.guardarProductoModal.mostrarProductos(); // Llamar al método mostrarProductos() en el componente hijo
      } catch (error) {
        swal("Error", error, "error")
      }
    },
    mostrarToast(mensaje, color) {
      this.toastMessage = mensaje;
      this.toastColor = color;
      this.showToast = true;
      setTimeout(() => {
        this.showToast = false;
      }, 3000);
    },
    async eliminarProducto(id) {
      try {
        const response = await fetch(`http://192.168.0.2:550/api/v1/Catalog/${id}`, {
          method: 'DELETE',
        });

        if (!response.ok) {
          swal("Error", "No Se Puede Eliminar El Producto", "error")
        }
        swal("Producto Eliminado", "El Producto Se Elimino Correctamente", "success")
        const card = document.getElementById(`card-${id}`);
        card.classList.add('animated-card-destroy'); // Agrega la clase para la animación de destrucción
        setTimeout(() => {
          this.mostrarProductos();
        }, 500);
      } catch (error) {
        swal("Error", error, "error")
      }
    },
    confirmarEliminacion(id) {
      swal({
        title: 'Confirmación',
        text: '¿Estás Seguro de que Quieres Eliminar este Producto?',
        icon: 'warning',
        buttons: {
          cancel: {
            text: "No",
            value: null,
            visible: true,
            className: "",
            closeModal: true,
          },
          confirm: {
            text: "Si",
            value: true,
            visible: true,
            className: "",
            closeModal: true
          }
        }
      }).then((result) => {
        if (result) {
          this.eliminarProducto(id);
        } else {
          // swal("Error", "No Se Puede Eliminar El Producto", "error")
        }
      });
    },
    actualizarProductos() {
      this.mostrarProductos();
    },
    async buscarProductoId() {
      if (this.searchById === "") {
        this.mostrarProductos();
      } else {
        try {
          const response = await fetch(`http://192.168.0.2:550/api/v1/Catalog/${this.searchById}`);
          if (!response.ok) {
            return;
          }
          const data = await response.json();
          this.products = [data];
        } catch (error) {
          this.mostrarToast('No Se Pudo Buscar El Producto Por ID', 'error');
        }
      }
    },
    async buscarProductoCategoria() {
      if (this.searchByCategory === "") {
        this.mostrarProductos();
      } else {
        try {
          const response = await fetch(`http://192.168.0.2:550/api/v1/Catalog/GetProductByCategory/${this.searchByCategory}`);
          if (!response.ok) {
            return;
          }
          const data = await response.json();
          this.products = data;
        } catch (error) {
          this.mostrarToast('No Se Pudo Buscar El Producto Por Categoría', 'error');
        }
      }
    },
    abrirModal(producto) {
      this.mostrarModalEdicion = true;
      const precioConvertido = parseFloat(producto.precio);
      this.productoSeleccionado.id = producto.id;
      this.productoSeleccionado.nombre = producto.nombre;
      this.productoSeleccionado.categoria = producto.categoria;
      this.productoSeleccionado.descripcion = producto.descripcion;
      this.productoSeleccionado.resumen = producto.resumen;
      this.productoSeleccionado.imagenArchivo = producto.imagenArchivo;
      this.productoSeleccionado.precio = precioConvertido.toFixed(2);
    },
    // Agrega el método de restablecerModal() para restablecer las variables de edición del producto
    restablecerModal() {
      this.mostrarModalEdicion = false;
      this.productoSeleccionado.id = '';
      this.productoSeleccionado.nombre = '';
      this.productoSeleccionado.categoria = '';
      this.productoSeleccionado.descripcion = '';
      this.productoSeleccionado.resumen = '';
      this.productoSeleccionado.imagenArchivo = '';
      this.productoSeleccionado.precio = '';
    },
  },
  computed: {
    searchIcon() {
      return this.searchByCategory || this.searchById ? 'close-circle' : 'search';
    },
  },
});
</script>

<style scoped>
.animated-card {
  animation: fade-in 0.5s ease-in-out;
}

@keyframes fade-in {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }

  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.hover-scale {
  transition: transform 0.3s ease-in-out;
}

.hover-scale:hover {
  transform: scale(1.05);
}

hr {
  margin-top: 10px;
  margin-bottom: 10px;
  border: 0;
  border-top: 1px solid #65233d;
}

.hover-elevate {
  transition: transform 0.3s ease-in-out;
}

.hover-elevate:hover {
  transform: translateY(-10px);
}


.animated-row {
  animation: fade-in 0.5s ease-in-out;
}

.animated-column {
  animation: fade-in 0.5s ease-in-out;
  animation-delay: 0.2s;
  /* Agrega un retraso de animación para los elementos de columna */
}

@keyframes fade-in {
  0% {
    opacity: 0;
    transform: translateX(-10px);
  }

  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

.animated-card-destroy {
  animation: destroy-card 0.5s ease-in-out;
  opacity: 0;
  transform: scale(0.8);
}

@keyframes destroy-card {
  0% {
    opacity: 1;
    transform: scale(1);
  }

  100% {
    opacity: 0;
    transform: scale(0.8);
  }
}

.typewriter-effect {
  overflow: hidden;
  /* Oculta el contenido que se va a escribir */
  white-space: nowrap;
  /* Evita que el texto se divida en múltiples líneas */
  animation: typing 3s steps(30) infinite;
}

@keyframes typing {
  from {
    width: 0;
  }

  to {
    width: 100%;
  }
}</style>
