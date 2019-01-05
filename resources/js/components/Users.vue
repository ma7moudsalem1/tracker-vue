<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users List</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal">
                                <i class="fa fa-plus"></i>
                            </button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody><tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Action</th>
                            </tr>
                            <tr v-for="user in users.data" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{ user.type | upText }}</td>
                                <td>
                                    <a href="#" class="btn btn-primary" @click.prevent="editModal(user)">
                                        <i class="fa fa-edit"></i>
                                    </a>
                                    <a href="#" class="btn btn-danger" @click.prevent="deleteUser(user.id)">
                                        <i class="fa fa-trash"></i>
                                    </a>
                                </td>
                            </tr>

                            </tbody></table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                </div>
                <!-- /.card -->
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewModal" tabindex="-1" role="dialog" aria-labelledby="addNewModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-if="editMode" class="modal-title" id="addNewModalLabel">Edit User</h5>
                        <h5 v-else="editMode" class="modal-title" id="addNewModalLabel">Add New User</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser() : createUser()">
                        <div class="modal-body">

                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name" placeholder="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.email" type="email" name="email" placeholder="Email address"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>

                            <div class="form-group">
                                <textarea  v-model="form.bio" name="bio" placeholder="Short bio (Optional)"
                                           class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }" id="bio"
                                           cols="30" rows="3"></textarea>

                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <select v-model="form.type" name="type"
                                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }" id="type">
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password" placeholder="Password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>



                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button v-if="editMode" type="submit" class="btn btn-primary">Update</button>
                            <button v-else="editMode" type="submit" class="btn btn-success">Create</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        data(){
            return {
                users: {},
                editMode: false,
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                })
            }
        },
        methods: {
            newModal(){
                this.form.reset();
                this.editMode = false;
                $('#addNewModal').modal('show');
            },
            editModal(user){
                this.form.reset();
                this.editMode = true;
                $('#addNewModal').modal('show');
                this.form.fill(user);
            },
            getResults(page = 1) {
                axios.get('/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            loadUsers(){
                axios.get("/user").then(({data}) => (this.users = data));
            },
            createUser(){
                this.form.post('/user').then(() => {
                    Fire.$emit('dataLoaded');
                    $('#addNewModal').modal('hide');
                    toast({
                        type: 'success',
                        title: 'User created successfully'
                    });
                }).catch();

            },
            updateUser(){
                this.form.patch('/user/'+this.form.id)
                    .then(() => {
                        Fire.$emit('dataLoaded');
                        $('#addNewModal').modal('hide');
                        toast({
                            type: 'success',
                            title: 'User Updared successfully'
                        });
                    })
                    .catch();
            }
            ,
            deleteUser(id){
                    Swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                        if(result.value){
                            this.form.delete('/user/' + id).then(() => {
                                Fire.$emit('dataLoaded');
                                Swal(
                                    'Deleted!',
                                    'Your file has been deleted.',
                                    'success'
                                )

                            }).catch(() => {
                                Swal(
                                    'Failed!',
                                    'Something went wrong.',
                                    'warning'
                                )
                            });
                        }

                    })
            }
        }
        ,
        created() {
            this.loadUsers();
            Fire.$on('dataLoaded', () => {
                this.loadUsers();
            });
        }
    }
</script>
