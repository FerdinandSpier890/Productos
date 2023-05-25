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
      <ion-row class="d-flex align-center justify-center">
        <ion-col cols="12" md="6">
          <ion-item>
            <ion-label position="floating">Buscar por ID</ion-label>
            <ion-input v-model="searchById" @input="buscarProductoId"></ion-input>
            <ion-icon slot="start" :icon="searchIcon"></ion-icon>
          </ion-item>
        </ion-col>
        <ion-col cols="12" md="6">
          <ion-item>
            <ion-label position="floating">Buscar por Categoría</ion-label>
            <ion-input v-model="searchByCategory" @input="buscarProductoCategoria"></ion-input>
            <ion-icon slot="start" :icon="searchIcon"></ion-icon>
          </ion-item>
        </ion-col>
      </ion-row>
      <ion-row class="d-flex align-center justify-center">
        <ion-col v-for="producto in products" :key="producto.id" size-md="3">
          <ion-card class="mx-auto align-center" max-width="250" style="border: 5px solid #65233d" elevation="10">
            <ion-img :src="producto.imagenArchivo" height="200px" />
            <ion-card-header class="font-weight-bold">
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
        const response = await fetch('https://localhost:44384/api/v1/Catalog');
        if (!response.ok) {
          this.mostrarToast('No Se Pudo Mostrar Los Productos', 'error');
        }
        const data = await response.json();
        this.products = data;
        this.$refs.guardarProductoModal.mostrarProductos(); // Llamar al método mostrarProductos() en el componente hijo
      } catch (error) {
        this.mostrarToast('No Se Pudo Obtener Los Productos', 'error');
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
        const response = await fetch(`https://localhost:44384/api/v1/Catalog/${id}`, {
          method: 'DELETE',
        });

        if (!response.ok) {
          swal("Error", "No Se Puede Eliminar El Producto", "error")
        }
        swal("Producto Eliminado", "El Producto Se Elimino Correctamente", "success")
        this.mostrarProductos();
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
      this.productoSeleccionado.precio = precioConvertido

    },
    obtenerProductos() {
      console.log('Método "mostrarProductos" ejecutado');
      // Lógica para obtener la lista actualizada de productos
      this.products = mostrarProductos(); // Asegúrate de que esta función obtenga los productos actualizados correctamente
    }

  },
});
</script>

<style scoped>
.btn-success {
  background-color: #4caf50 !important;
  color: #fff;
}

.btn-error {
  background-color: #f44336 !important;
  color: #fff;
}

ion-fab-button::part(native) {
  background-color: #146b63;
  border-radius: 15px;
  box-shadow: 0px 1px 2px 0px rgba(0, 0, 0, 0.3), 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
  color: black;
}

ion-fab-button::part(native):hover::after {
  background-color: #a3e681;
}

ion-fab-button::part(native):active::after {
  background-color: #87d361;
}
</style>
