<template>
  <ion-page>
    <ion-header>
      <ion-toolbar style="background-color: #6f9e4a;">
        <ion-title style="font-family: 'Minecraft', sans-serif;">Lista de Productos</ion-title>
        <ion-buttons slot="end">
          <ion-button style="font-family: 'Minecraft', sans-serif;" @click="nuevoProducto = true">
            <ion-icon name="add"></ion-icon> Agregar Producto
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    <ion-content
      style="background-image: url('https://addons-media.operacdn.com/media/CACHE/images/themes/95/72895/1.0-rev1/images/93ccab91-6074-4c33-8e46-e595084a0d4e/0dfd2182106b1095cc52bd11ba596698.jpg');">
      <ion-row class="d-flex align-center justify-center animated-row fade-in">
        <ion-col cols="12" md="6" class="animated-column fade-in">
          <ion-item>
            <ion-label position="floating" style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Buscar por ID</ion-label>
            <ion-input v-model="searchById" @input="buscarProductoId"
              style="font-family: 'Minecraft', sans-serif;"></ion-input>
            <ion-icon slot="start" :icon="searchIcon" style="color: #6f9e4a;"></ion-icon>
          </ion-item>
        </ion-col>
        <ion-col cols="12" md="6" class="animated-column fade-in">
          <ion-item>
            <ion-label position="floating" style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Buscar por Categoría</ion-label>
            <ion-input v-model="searchByCategory" @input="buscarProductoCategoria"
              style="font-family: 'Minecraft', sans-serif;"></ion-input>
            <ion-icon slot="start" :icon="searchIcon" style="color: #6f9e4a;"></ion-icon>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row class="d-flex align-center justify-center">
        <ion-col v-for="producto in products" :key="producto.id" size-md="3">
          <ion-card :id="'card-' + producto.id" class="animated-card fade-in mx-auto align-center" max-width="auto"
            style="border: 5px solid #6f9e4a; border-radius: 10px; background-color: #6f9e4a; color: #FF0000"
            elevation="10">
            <ion-img :src="producto.imagenArchivo" height="200px" style="border-radius: 10px;" />
            <ion-card-header class="font-weight-bold typewriter-effect"
              style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">
              <ion-card-title style="color: #FFFFFF">{{ producto.nombre }}</ion-card-title>
            </ion-card-header>
            <hr style="border-color: #3a5b85;" />
            <ion-card-content>
              <ion-card-subtitle>
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">ID:
                  {{ producto.id
                  }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">Nombre del Producto: {{
                    producto.nombre }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">Categoría del Producto:
                  {{
                    producto.categoria }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">Resumen del Producto: {{
                    producto.resumen }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">Descripción del Producto:
                  {{
                    producto.descripcion }}</ion-card-text> <br />
                <ion-card-text class="font-weight-bold"
                  style="font-family: 'Minecraft', sans-serif; color: #FFFFFF; font-size: 18px;">Precio del Producto: ${{
                    producto.precio.toFixed(2) }}</ion-card-text>
              </ion-card-subtitle>
              <ion-row class="ion-align-items-center ion-justify-content-center">
                <ion-col>
                  <ion-button expand="full" color="success" @click="abrirModal(producto)"
                    style="font-family: 'Minecraft', sans-serif;">
                    Editar
                  </ion-button>
                </ion-col>
                <ion-col>
                  <ion-button expand="full" color="danger" @click="confirmarEliminacion(producto.id)"
                    style="font-family: 'Minecraft', sans-serif;">
                    Eliminar
                  </ion-button>
                </ion-col>
              </ion-row>
            </ion-card-content>
          </ion-card>
        </ion-col>
      </ion-row>
      <!-- Componente del formulario de guardar producto -->
      <GuardarProductoModal v-bind:nuevoProducto="nuevoProducto" formTitle="Nuevo Producto" @dismiss="resetModal"
        @actualizarProductos="obtenerProductos" />

      <EditarProductoModal :productoSeleccionado="productoSeleccionado" :editarProductoModal="mostrarModalEdicion"
        @dismiss="resetModal" :is-open="mostrarModalEdicion" />
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
    //this.musicaFondo();
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
        const response = await fetch('https://localhost:44384/api/v1/Catalog');
        if (!response.ok) {
          swal("Error", "No Se Puede Obtener los Productos", "error")
        }
        const data = await response.json();
        const audio = new Audio('../sounds/xp.mp3');
        //audio.currentTime = 0;
        audio.play();
        this.products = data;
        this.$refs.guardarProductoModal.mostrarProductos(); // Llamar al método mostrarProductos() en el componente hijo
      } catch (error) {
        swal("Error", error, "error")
      }
    },
    async eliminarProducto(id) {
      try {
        const response = await fetch(`https://localhost:44384/api/v1/Catalog/${id}`, {
          method: 'DELETE',
        });

        if (!response.ok) {
          swal("Error", "No Se Puede Eliminar El Producto", "error")
        }
        const audio = new Audio('../sounds/netherenter.mp3');
        audio.currentTime = 0;
        audio.play();
        swal("Producto Eliminado", "El Producto Se Elimino Correctamente", "success")
        const card = document.getElementById(`card-${id}`);
        card.classList.add('burn-effect');

        setTimeout(() => {
          this.mostrarProductos();
        }, 5000);
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
          const response = await fetch(`https://localhost:44384/api/v1/Catalog/${this.searchById}`);
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
          const response = await fetch(`https://localhost:44384/api/v1/Catalog/GetProductByCategory/${this.searchByCategory}`);
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
    musicaFondo() {
      const audio = new Audio('../sounds/sweden.mp3');
      audio.play();
    }
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
}

.animated-card {
  position: relative;
  transition: box-shadow 0.3s ease-in-out;
}

.animated-card:hover {
  animation: star-glow 1s infinite alternate;
}

@keyframes star-glow {
  0% {
    box-shadow: 0 0 100px 0 #6f9e4a;
  }

  50% {
    box-shadow: 0 0 200px 0 #6f9e4a;
  }

  100% {
    box-shadow: 0 0 100px 0 #6f9e4a;
  }
}

@keyframes enlarge-disappear {
  0% {
    opacity: 1;
    transform: scale(1);
  }

  100% {
    opacity: 0;
    transform: scale(3) rotate(360deg);
  }
}

.animated-card-remove {
  animation: enlarge-disappear 1s ease-in-out;
}

.ion-fab {
  animation: fade-in 0.5s ease-in-out;
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}

@keyframes burn-animation {
  0% {
    opacity: 1;
    transform: scale(1);
    background-color: #6f9e4a;
    color: #7833ae;
  }

  50% {
    opacity: 0.5;
    transform: scale(1.1);
    background-color: #5e1c9e;
    color: #FFFFFF;
  }

  100% {
    opacity: 0;
    transform: scale(1.2);
    background-color: #3f1187;
    color: #FFFFFF;
  }
}

.burn-effect {
  animation: burn-animation 5s linear forwards;
  animation-delay: 0.2s;
  /* Agrega un retraso de animación para que se muestre la animación después de eliminar el producto */
}</style>
