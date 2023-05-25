<template>
    <ion-modal :is-open="mostrarModalEdicion" @dismiss="cerrarModal">
        <ion-header>
            <ion-toolbar>
                <ion-title>Editar Producto</ion-title>
            </ion-toolbar>
        </ion-header>
        <ion-content>
            <ion-list>
                <ion-item>
                    <ion-img :src="productoSeleccionado.imagenArchivo" height="10px" width="10px" />
                </ion-item>
                <ion-item>
                    <ion-label position="floating">Nombre</ion-label>
                    <ion-input v-model="productoSeleccionado.nombre" type="text"></ion-input>
                </ion-item>
                <ion-item>
                    <ion-label position="floating">Categoría</ion-label>
                    <ion-input v-model="productoSeleccionado.categoria" type="text"></ion-input>
                </ion-item>
                <ion-item>
                    <ion-label position="floating">Resumen</ion-label>
                    <ion-input v-model="productoSeleccionado.resumen"></ion-input>
                </ion-item>
                <ion-item>
                    <ion-label position="floating">Descripción</ion-label>
                    <ion-input v-model="productoSeleccionado.descripcion"></ion-input>
                </ion-item>

                <ion-item>
                    <ion-label position="floating">Imagen</ion-label>
                    <ion-input v-model="productoSeleccionado.imagenArchivo" type="text"></ion-input>
                </ion-item>
                <ion-item>
                    <ion-label position="floating">Precio</ion-label>
                    <ion-input v-model="productoSeleccionado.precio" type="number"></ion-input>
                </ion-item>
            </ion-list>
            <ion-footer>
                <ion-toolbar>
                    <ion-buttons slot="start">
                        <ion-button color="danger" @click="cerrarModal">
                            Cancelar
                        </ion-button>
                    </ion-buttons>
                    <ion-buttons slot="end">
                        <ion-button color="primary" @click="guardarCambios">
                            Guardar
                        </ion-button>
                    </ion-buttons>
                </ion-toolbar>
            </ion-footer>
        </ion-content>
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
        productoSeleccionado: {
            type: Object,
            required: true
        },
        editarProductoModal: {
            type: Boolean,
            required: true
        }
    },

    data() {
        return {
            editarProducto: false
        };
    },
    methods: {
        abrirModal() {
            this.$emit('update:mostrarModalEdicion', true);
            // this.editarProductoModal = true;
        },

        cerrarModal() {
            this.$emit('dismiss');
            this.$emit('update:mostrarModalEdicion', false);
            // this.editarProductoModal = false;
            this.formTitle = '';
            this.productoSeleccionado.nombre = '';
            this.productoSeleccionado.categoria = '';
            this.productoSeleccionado.resumen = '';
            this.productoSeleccionado.descripcion = '';
            this.productoSeleccionado.imagenArchivo = '';
            this.productoSeleccionado.precio = '';
        },

        async guardarCambios() {
            try {
                // Convertir el valor de precio a un número decimal
                const precioConvertido = parseFloat(this.productoSeleccionado.precio);

                // Asignar el valor convertido al formulario
                this.productoSeleccionado.precio = precioConvertido;

                const response = await fetch(`https://localhost:44384/api/v1/Catalog/${this.productoSeleccionado.id}`, {
                    method: "PUT",
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(this.productoSeleccionado),
                });
                if (response.ok) {
                    //this.products = this.products.filter((producto) => producto.id !== id);
                    swal("Actualización Exitosa", "El Producto Se Actualizó Correctamente", "success");
                    //location.reload();
                    // Actualizar la lista de productos después de actualizar
                    this.$emit('update:editarProductoModal', false);
                    this.$emit('dismiss');

                } else {
                    swal("Error", "No Se Puede Actualizar El Producto", "error");
                    console.error('Error al actualizar el producto', response);
                }
            } catch (error) {
                swal("Error", error, "error");
                console.error('Error al actualizar el producto', error);
            }
        }
    }
});
</script>
  