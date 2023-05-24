<template>
    <ion-modal :is-open="nuevoProducto" @dismiss="cerrarFormulario">
      <ion-header>
        <ion-toolbar>
          <ion-title>{{ formTitle }}</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="cerrarFormulario">
              <ion-icon name="close"></ion-icon>
            </ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
  
      <ion-content>
        <ion-card>
          <ion-card-header>
            <ion-card-title>{{ formTitle }}</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <ion-item>
              <ion-label position="floating">Nombre del Producto</ion-label>
              <ion-input v-model="nombre" required></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Categoría del Producto</ion-label>
              <ion-input v-model="categoria" required></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Resumen del Producto</ion-label>
              <ion-input v-model="resumen" required></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Descripción del Producto</ion-label>
              <ion-input v-model="descripcion" required></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Imagen del Producto</ion-label>
              <ion-input v-model="imagenArchivo" required type="url"></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Precio del Producto</ion-label>
              <ion-input v-model="precio" required type="number"></ion-input>
            </ion-item>
          </ion-card-content>
        </ion-card>
  
        <div class="ion-padding">
          <ion-button expand="block" color="secondary" @click="cerrarFormulario">Cancelar</ion-button>
          <ion-button expand="block" color="primary" @click="guardarProducto">Guardar</ion-button>
        </div>
      </ion-content>
  
      <ion-toast :is-open="showToast" :message="toastMessage" :color="toastColor" position="bottom" :duration="3000"></ion-toast>
    </ion-modal>
  </template>
  
  <script>
  import { IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonButton, IonItem, IonLabel, IonInput, IonModal, IonToast } from '@ionic/vue';
  import { defineComponent } from 'vue';
  
  export default defineComponent({
    components: {
      IonCard,
      IonCardHeader,
      IonCardTitle,
      IonCardContent,
      IonButton,
      IonItem,
      IonLabel,
      IonInput,
      IonModal,
      IonToast
    },
    props: {
      nuevoProducto: {
        type: Boolean,
        required: true
      },
      formTitle: {
        type: String,
        required: true
      },
    },
    data() {
      return {
        nombre: '',
        categoria: '',
        resumen: '',
        descripcion: '',
        imagenArchivo: '',
        precio: '',
        showToast: false,
        toastMessage: '',
        toastColor: ''
      };
    },
    methods: {
      cerrarFormulario() {
        this.nuevoProducto = false;
        this.$emit('dismiss');
      },
      resetFormulario() {
        this.nombre = '';
        this.categoria = '';
        this.resumen = '';
        this.descripcion = '';
        this.imagenArchivo = '';
        this.precio = '';
      },
      async guardarProducto() {
        const precioConvertido = this.convertirPrecio(this.precio);
  
        const producto = {
          nombre: this.nombre,
          categoria: this.categoria,
          resumen: this.resumen,
          descripcion: this.descripcion,
          imagenArchivo: this.imagenArchivo,
          precio: precioConvertido
        };
  
        const response = await fetch("https://localhost:44384/api/v1/Catalog", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(producto),
        });
  
        if (!response.ok) {
          this.mostrarToast('No se pudo registrar el producto. Inténtalo de nuevo.', 'danger');
          return;
  
        } else {
          this.mostrarToast('Se registró el producto exitosamente.', 'success');
        }
  
        console.log(response)
  
        this.nuevoProducto = false;
        this.resetFormulario();
        this.cerrarFormulario();
      },
      convertirPrecio(valor) {
        if (!valor) {
          return null;
        }
        return parseFloat(valor);
      },
      mostrarToast(mensaje, color) {
        this.toastMessage = mensaje;
        this.toastColor = color;
        this.showToast = true;
        setTimeout(() => {
          this.showToast = false;
        }, 3000);
      },
    }
  });
  </script>
  