<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">
                        <div class="row">
                            <div class="col-sm-6">
                                <h3>Listado de productos</h3>
                            </div>
                            <div class="col-sm-6 text-right">
                                <button @click="showModal" type="button" class="btn btn-primary">
                                    Nuevo produto
                                </button>
                            </div>
                        </div>
                        
                        
                    </div>

                    <div class="card-body">
                        <table class="table table-striped table-hover table-bordered">
                            <thead class="thead-dark">
                                <tr>
                                    <th>#</th>
                                    <th>NOMBRES</th>
                                    <th>DESCRIPCION</th>
                                    <th>PRECIO</th>
                                    <th>ESTADO</th>
                                    <th></th>
                                </tr>

                            </thead>
                            <tbody>
                                <tr v-for="prod in products" :key="prod.id">
                                    <td>{{ prod.id }}</td>
                                    <td>{{ prod.name }}</td>
                                    <td>{{ prod.description }}</td>
                                    <td>{{ prod.price }}</td>
                                    <td>
                                        <span v-if="prod.enabled == 1" class="badge badge-success">Activo</span>
                                        <span v-else class="badge badge-danger">Inactivo</span>
                                    </td>
                                    <td>
                                        <button title="Editar Producto" class="btn btn-secondary btn-sm" @click="showModal(prod)"> Editar</button>
                                        <button title="Eliminar Producto" class="btn btn-danger btn-sm" @click="destroy(prod.id)"> Eliminar</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
        <!-- The Modal -->
        <div class="modal" :class="{mostrar:modal}">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="exampleModalLabel">{{ modalTitle }}</h5>
                    <button @click="closeModal()" type="button" class="close" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="form-group row">
                            <label for="name" class="col-sm-2 col-form-label">Nombre</label>
                            <div class="col-sm-10">
                                <input type="text" class="form-control" id="name" value="" v-model="product.name">
                                <span v-for="err in errors.name" :key="err" class="text-danger">{{ err }}</span>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="description" class="col-sm-2 col-form-label">Descripción</label>
                            <div class="col-sm-10">
                                <input type="text" class="form-control" id="description" v-model="product.description">
                                <span v-for="err in errors.description" :key="err" class="text-danger">{{ err }}</span>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="price" class="col-sm-2 col-form-label">Precio</label>
                            <div class="col-sm-10">
                                <input type="number" step="any" class="form-control" id="price" v-model="product.price">
                                <span v-for="err in errors.price" :key="err" class="text-danger">{{ err }}</span>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="enabled" class="col-sm-2 col-form-label">Activo</label>
                            <div class="col-sm-10">
                                <select name="enabled" id="enabled" class="form-control" v-model="product.enabled">
                                    <option value="1">SI</option>
                                    <option value="0">NO</option>
                                </select>
                            </div>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="closeModal()" >Cerrar</button>
                    <button type="button" class="btn btn-primary" @click="guardar();">Guardar</button>
                </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                modalTitle:'',
                products: [],
                product:{
                    id: 0,
                    name: '',
                    description: '',
                    price: 0.0,
                    enabled: '1',
                },
                modal: false,
                errors:[],
            }
        },
        mounted() {
            console.log('Component mounted.')
        },
        methods: {
            async getProducts (){
                axios.get('/product')
                    .then(response => (this.products = response.data))
            },
            async destroy(id){
                if(confirm("¿Confirma eliminar el producto?")){
                    axios.delete('/product/'+id, this.product)
                    .then(response => {
                        this.getProducts();
                        alert("Eliminado exitosamente");
                    })
                    .catch(error => {
                        this.errors = error.response.data.errors;
                    })
                }
                
            },
            async guardar (){
                if(this.product.id == 0){
                    axios.post('/product', this.product)
                    .then(response => {
                        this.closeModal();
                        this.getProducts();
                        alert("Registrado con exito");
                    })
                    .catch(error => {
                        this.errors = error.response.data.errors;
                    })
                }else{
                    axios.put('/product/'+this.product.id, this.product)
                    .then(response => {
                        this.closeModal();
                        this.getProducts();  
                        alert("Actualizado con exito");
                    })
                    .catch(error => {
                        this.errors = error.response.data.errors;
                    })
                }
            },
            showModal(data = {}){
                if(data.id){
                    this.modalTitle = "Editar Producto";
                    this.product.id = data.id;
                    this.product.name = data.name;
                    this.product.description = data.description;
                    this.product.price = data.price;
                    this.product.enabled = data.enabled;
                }else{
                    this.resertData();
                    this.modalTitle = "Agregar Producto";
                }
                this.modal = true;
            },
            closeModal(){
                this.modal = false;
                this.errors = [];
                this.resertData();
            },
            resertData: function(){
                this.modalTitle = "";
                this.product.id = 0;
                this.product.name = '';
                this.product.description = "";
                this.product.price = 0.00;
                this.product.enabled = 1;
            },
        },
        created() {
            this.getProducts();
        }
    }
</script>

<style>
    .mostrar{
        display: list-item;    
        opacity: 1;
        background: rgba(44, 38, 75, 0.849);
    }
</style>