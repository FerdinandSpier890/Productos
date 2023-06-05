<template>
    <ion-modal :is-open="mostrarModalEdicion" @dismiss="cerrarModal">
        <ion-header>
            <ion-toolbar style="background-color: #20262E;">
                <ion-title style="color: #fff;">Editar Producto</ion-title>
                <ion-buttons slot="end">
                    <ion-button @click="cerrarModal">
                        <ion-icon name="close"></ion-icon>
                    </ion-button>
                </ion-buttons>
            </ion-toolbar>
        </ion-header>
        <ion-content style="background-color: #20262E">
            <ion-card>
                <ion-card-content>
                    <ion-item>
                        <ion-img :src="productoSeleccionado.imagenArchivo" height="100" width="100" />
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Nombre</ion-label>
                        <ion-input v-model="productoSeleccionado.nombre" type="text"></ion-input>
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Categoría</ion-label>
                        <ion-input v-model="productoSeleccionado.categoria" type="text"></ion-input>
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Resumen</ion-label>
                        <ion-input v-model="productoSeleccionado.resumen"></ion-input>
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Descripción</ion-label>
                        <ion-input v-model="productoSeleccionado.descripcion"></ion-input>
                    </ion-item>

                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Imagen</ion-label>
                        <ion-input v-model="productoSeleccionado.imagenArchivo" type="text"></ion-input>
                    </ion-item>
                    <ion-item>
                        <ion-label position="floating"
                            style="font-family: 'Minecraft', sans-serif; color: #FFFFFF">Precio</ion-label>
                        <ion-input v-model="productoSeleccionado.precio" type="number"></ion-input>
                    </ion-item>
                </ion-card-content>
            </ion-card>
            <ion-footer style="background-color: #20262E;">
                <ion-toolbar>
                    <ion-buttons slot="start">
                        <ion-button color="white" @click="cerrarModal"
                            style="font-family: 'Minecraft', sans-serif; background-color: #20262E; border-radius: 10px;">
                            <i class="fas fa-times fa-xl" style="margin-right: 10px;"></i> Cancelar
                        </ion-button>
                    </ion-buttons>
                    <ion-buttons slot="end">
                        <ion-button color="white" @click="guardarCambios"
                            style="font-family: 'Minecraft', sans-serif; background-color: #20262E; border-radius: 10px;">
                            <i class="fas fa-save fa-xl" style="margin-right: 10px;"></i> Guardar
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
        validarCampos() {
            if (
                this.productoSeleccionado.nombre.trim() === "" ||
                this.productoSeleccionado.categoria.trim() === "" ||
                this.productoSeleccionado.resumen.trim() === "" ||
                this.productoSeleccionado.descripcion.trim() === "" ||
                this.productoSeleccionado.imagenArchivo.trim() === "" ||
                this.productoSeleccionado.precio.trim() === ""
            ) {
                swal("Campos Vacíos", "Por favor, complete todos los campos", "warning");
                return false;
            }

            if (!this.validarURL(this.productoSeleccionado.imagenArchivo)) {
                swal("URL Inválida", "La URL de la imagen no es válida", "warning");
                return false;
            }

            if (!this.validarNumero(this.productoSeleccionado.precio)) {
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
                if (!this.validarCampos()) {
                    return;
                } else {
                    // Convertir el valor de precio a un número decimal
                    const precioConvertido = parseFloat(this.productoSeleccionado.precio);

                    // Asignar el valor convertido al formulario
                    this.productoSeleccionado.precio = precioConvertido;

                    const response = await fetch(`http://192.168.0.2:550/api/v1/Catalog/${this.productoSeleccionado.id}`, {
                        method: "PUT",
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify(this.productoSeleccionado),
                    });
                    if (response.ok) {
                        //this.products = this.products.filter((producto) => producto.id !== id);
                        const sound = new Audio('../sounds/xp.mp3');
                        sound.play();
                        swal("Actualización Exitosa", "El Producto Se Actualizó Correctamente", "success");
                        setTimeout(() => {
                            location.reload();
                        }, 6000);
                        // Actualizar la lista de productos después de actualizar
                        this.$emit('update:editarProductoModal', false);
                        this.$emit('dismiss');

                    } else {
                        swal("Error", "No Se Puede Actualizar El Producto", "error");
                        console.error('Error al actualizar el producto', response);
                    }
                }
            } catch (error) {
                swal("Error", error, "error");
                console.error('Error al actualizar el producto', error);
            }
        }
    }
});
</script>
<style scoped>
/* Estilos generales */
body {
    background-color: #c4c4c4;
    /* Cambia el color de fondo aquí */
}

ion-label,
ion-input {
    font-family: 'Minecraft', sans-serif;
}

/* Estilos específicos del componente */
.ion-modal {
    font-family: 'Minecraft', sans-serif;
}

.ion-header {
    background-color: #3a5b85;
    /* Cambia el color del encabezado aquí */
}

.ion-header ion-title {
    color: #fff;
    /* Cambia el color del texto del título aquí */
}

.ion-content {
    background-color: #c4c4c4;
    /* Cambia el color del contenido aquí */
}

.ion-item {
    background-color: #fff;
    /* Cambia el color de los elementos de item aquí */
    border-radius: 8px;
    margin-bottom: 12px;
}

.ion-item ion-label {
    color: #3a5b85;
    /* Cambia el color del texto de etiqueta aquí */
}

.ion-item ion-input {
    color: #3a5b85;
    /* Cambia el color del texto de entrada aquí */
}

.ion-footer {
    background-color: #3a5b85;
    /* Cambia el color del pie de página aquí */
}

.ion-footer ion-toolbar {
    justify-content: space-between;
}

.ion-footer ion-button {
    color: #fff;
    /* Cambia el color del texto del botón aquí */
}</style>