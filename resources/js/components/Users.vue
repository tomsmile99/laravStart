<template>
    <div class="container p-4">
        <div class="row">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>
                <div class="card-tools">
                    <button class="btn btn-success" @click="addNewUserModel">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registered At</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at | myDate}}</td>
                      <td>
                          <a href="" @click="editUserModel(user)">
                              <a href="#" class="btn btn-warning btn-sm"><i class="fa fa-edit"></i> Edit</a>
                          </a>

                          <a href="#" @click="deleteUser(user.id)">
                              <a href="#" class="btn btn-danger btn-sm"><i class="fa fa-trash"></i> Delet</a>
                          </a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNewUser" tabindex="-1" role="dialog" aria-labelledby="AddNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editmode" class="modal-title" id="AddNewLabel">Add New Users</h5>
                        <h5 v-show="editmode" class="modal-title" id="AddNewLabel">Edit Users</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <label>Name : </label>
                                <input v-model="form.name" type="text" name="name"
                                    placeholder="Name"
                                    class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Email : </label>
                                <input v-model="form.email" type="email" name="email"
                                    placeholder="Email Address"
                                    class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Bio : </label>
                                <textarea v-model="form.bio" name="bio" id="bio"
                                    placeholder="Short bio for user (Optional)"
                                    class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>
                            <div class="form-group">
                                <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password" id="password"
                                    placeholder=""
                                    class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button v-show="!editmode" type="submit" class="btn btn-success"><i class="fas fa-save"></i> Create</button>
                            <button v-show="editmode" type="submit" class="btn btn-warning"><i class="fas fa-save"></i> Update</button>
                            <button type="button" class="btn btn-secondary" data-dismiss="modal"><i class="fas fa-window-close"></i> Close</button>
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
            return {
                editmode: false,
                users : {},
                form: new Form({
                    id:'',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : ''
                })
            }
        },
        methods: {
            updateUser(){
                //console.log('Editing data');
                this.$Progress.start();
                this.form.put('api/user/' +this.form.id)
                .then(() => {
                    // success
                    $('#addNewUser').modal('hide');
                    toast.fire({
                        icon: 'success',
                        title: 'Information has been updated'
                    })
                    this.$Progress.finish();
                    Fire.$emit('AfterCreate');
                })
                .catch(() => {
                    this.$Progress.fail();
                });
                this.$Progress.finish();
            },
            editUserModel(user){
                this.editmode = true;
                this.form.reset();
                $('#addNewUser').modal('show');
                this.form.fill(user);
            },
            addNewUserModel(){
                this.editmode = false;
                this.form.reset();
                $('#addNewUser').modal('show');
            },
            deleteUser(id){
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                        
                        // Send request to the server
                        if (result.value) {
                            this.$Progress.start();
                            this.form.delete('api/user/'+id).then(()=>{
                                toast.fire({
                                    icon: 'success',
                                    title: 'Your file has been deleted'
                                })
                                Fire.$emit('AfterCreate');
                            }).catch(()=>{
                                swal("Failed!", "There was something wronge.", "warning");
                            });
                            this.$Progress.finish();
                        }
                    })
                
            },
            loadUser(){
                axios.get("api/user").then(({ data }) => (this.users = data.data))
            },
            createUser() {
                this.$Progress.start();
                this.form.post('api/user')
                .then(() =>{
                    Fire.$emit('AfterCreate');
                    $('#addNewUser').modal('hide')
                    toast.fire({
                        icon: 'success',
                        title: 'Create Users in successfully'
                    })
                    
                })
                .catch(() =>{

                })
                this.$Progress.finish();
            }
        },
        created() {
            this.$Progress.start();
            this.loadUser();
            Fire.$on('AfterCreate',() => {
                this.loadUser();
            });
            this.$Progress.finish();
            //setInterval(() => this.loadUser(), 3000);
        }
    }
</script>
