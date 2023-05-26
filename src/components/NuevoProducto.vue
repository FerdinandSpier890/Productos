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
                <ion-card-content>
                    <ion-item>
                        <ion-img :src="imagenArchivo" height="200px" />
                    </ion-item>
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
                        <ion-input v-model="imagenArchivo" required></ion-input>
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating">Precio del Producto</ion-label>
                        <ion-input v-model="precio" required></ion-input>
                    </ion-item>
                </ion-card-content>
            </ion-card>
            <ion-footer>
                <ion-toolbar>
                    <ion-buttons slot="start">
                        <ion-button color="danger" @click="cerrarFormulario">Cancelar</ion-button>
                    </ion-buttons>
                    <ion-buttons slot="end">
                        <ion-button color="primary" @click="guardarProducto">Guardar</ion-button>
                    </ion-buttons>
                </ion-toolbar>
            </ion-footer>
        </ion-content>

        <ion-toast :is-open="showToast" :message="toastMessage" :color="toastColor" position="bottom"
            :duration="3000"></ion-toast>
    </ion-modal>
</template>
  
<script>
import { IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonButton, IonItem, IonLabel, IonInput, IonModal, IonToast } from '@ionic/vue';
import { defineComponent } from 'vue';
import swal from 'sweetalert';

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
            toastColor: '',
            products: [],

        };
    },
    methods: {
        cerrarFormulario() {
            this.$emit('dismiss');
            this.nuevoProducto = false;
            this.formTitle = '';
            this.nombre = '';
            this.categoria = '';
            this.resumen = '';
            this.descripcion = '';
            this.imagenArchivo = '';
            this.precio = '';
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
            if (this.validarCampos()) {
                const precioConvertido = this.convertirPrecio(this.precio);

                const producto = {
                    nombre: this.nombre,
                    categoria: this.categoria,
                    resumen: this.resumen,
                    descripcion: this.descripcion,
                    imagenArchivo: this.imagenArchivo,
                    precio: precioConvertido
                };

                try {
                    const response = await fetch("http://192.168.0.2:550/api/v1/Catalog/", {
                        method: "POST",
                        headers: {
                            "Content-Type": "application/json",
                        },
                        body: JSON.stringify(producto),
                    });

                    if (response.ok) {
                        swal("Registro Exitoso", "El Producto Se Registró Correctamente", "success");
                        this.cerrarFormulario();
                        location.reload();
                        this.$emit('actualizarProductos');
                        this.$emit('dismiss'); // Emitir evento 'dismiss' después de cerrar el formulario
                        this.mostrarProductos(); // Actualizar la lista de productos
                        //location.reload();
                    } else {
                        swal("Error", "No Se Puede Guardar El Producto", "error");
                    }

                    console.log(response);
                    this.nuevoProducto = false;
                    this.resetFormulario();
                    this.$emit('actualizarProductos');
                    this.cerrarFormulario();
                    this.mostrarProductos(); // Actualizar la lista de productos
                } catch (error) {
                    swal("Error", error, "error");
                }
            }
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
        async mostrarProductos() {
            try {
                const response = await fetch('http://192.168.0.2:550/api/v1/Catalog');
                if (!response.ok) {
                    swal("Error", "No Se Puede Mostrar Los Productos", "error")
                }
                const data = await response.json();
                this.products = data;
            } catch (error) {
                swal("Error", error, "error")
            }
        },
        actualizarProductos() {
            this.mostrarProductos();
        },
        validarCampos() {
            if (this.nombre.trim() === "" ||
                this.categoria.trim() === "" ||
                this.resumen.trim() === "" ||
                this.descripcion.trim() === "" ||
                this.imagenArchivo.trim() === "" ||
                this.precio.trim() === "") {
                swal("Campos Vacíos", "Por favor, complete todos los campos", "warning");
                return false;
            }

            if (!this.validarURL(this.imagenArchivo)) {
                swal("URL Inválida", "La URL de la imagen no es válida", "warning");
                return false;
            }

            if (!this.validarNumero(this.precio)) {
                swal("Precio Inválido", "El valor del precio no es válido", "warning");
                return false;
            }

            return true;
        },
        validarURL(url) {
            // Expresión regular para validar una URL
            const urlRegex = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/i;
            return urlRegex.test(url);
        },
        validarNumero(numero) {
            return !isNaN(parseFloat(numero)) && isFinite(numero);
        },
    }
});
</script>
<style scoped>

</style>