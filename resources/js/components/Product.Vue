<template>
    <div class="container">
        <div class="row mt-4">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Products Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add Products 
                      <i class="fas fa-folder-open fa-fw"></i>
                    </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Description</th>
                    <th>Quantity</th>                    
                    <th>Modify</th>
                  </tr>

                  
                  <tr v-for="product in products" :key="product.id"> 
                    <td>{{product.id}}</td>
                    <td>{{product.name}}</td>
                    <td>{{product.description}}</td>
                    <td>{{product.quantity}}</td>
                    <td>
                        <a href="#" @click="editModal(product)">
                            <i class="fa fa-edit blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteProduct(product.id)">
                            <i class="fa fa-trash red"></i>
                        </a>
                    </td>
                  </tr>
                  
                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" v-show="!editMode" id="addNew">Add New Product</h5>
                <h5 class="modal-title" v-show="editMode" id="addNew">Update product's Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editMode ? updateProduct(): createProduct()">
                <div class="modal-body">

                <div class="form-group">                
                    <input v-model="form.name" type="text" name="name"
                      placeholder="Name"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                </div>

                <div class="form-group">                
                    <input v-model="form.description" type="text" name="description"
                      placeholder="Description"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('description') }">
                    <has-error :form="form" field="description"></has-error>
                </div>

                <div class="form-group">                
                    <input v-model="form.quantity" type="text" name="quantity"
                      placeholder="Quantity"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('quantity') }">
                    <has-error :form="form" field="quantity"></has-error>
                </div>        

                </div>
                <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                </div>
              </form>
            </div>
          </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
          return{
            editMode: false,
            products : {},
            form: new Form({              
              name: '',  
              description: '',
              quantity: '',                                
            })
          }
        },
        methods: {
          updateProduct(){
            this.$Progress.start();
            //console.log('Editing data');
            this.form.put('api/product/'+this.form.id)
            .then(() =>{
              //success
              $('#addNew').modal('hide')
              Swal.fire(
                'Updated!',
                'Information has been updated.',
                'success'
              ) 
              Fire.$emit('AfterCreate')
              this.$Progress.finish();              

            })
            .catch(() =>{
              this.$Progress.fail(); 
            })
          },
          editModal(user){
            this.editMode = true;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(user);
          },
          newModal(){
            this.editMode = false;
            this.form.reset();
            $('#addNew').modal('show');
          },

          deleteProduct(id){
              Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).then((result) => {

                //send request to the serve
                if (result.value) {
                    this.form.delete('api/product/'+id).then(()=>{                  
                        Swal.fire(
                          'Deleted!',
                          'Your file has been deleted.',
                          'success'
                        ) 
                      Fire.$emit('AfterCreate')                
                    }).catch(()=> {
                      Swal("Failed","There was something wrong.", "warning");
                    });
                }
                
              })
          },
          loadProducts(){       
            axios.get('api/product').then(({ data }) => (this.products = data.data));
          },

          createProduct(){
            this.$Progress.start();

            this.form.post('api/product')
            .then(()=>{
                Fire.$emit('AfterCreate');

                $('#addNew').modal('hide')

                toast.fire({
                  type: 'success',
                  title: 'Product Created successfully'
                })

                this.$Progress.finish(); 

            })
            .catch(()=>{

            })


          }
        },
        created() {
            this.loadProducts();
            Fire.$on('AfterCreate',() => {
              this.loadProducts();
            })
            //setInterval(() => this.loadUsers(), 3000);
        }
    }
</script>