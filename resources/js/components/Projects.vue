<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Projects List</h3>

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
                                <th>User</th>
                                <th>Status</th>
                                <th>Action</th>
                            </tr>
                            <tr v-for="project in projects.data" :key="project.id">
                                <td>{{project.id}}</td>
                                <td>{{project.name}}</td>
                                <td>{{project.user.name}}</td>
                                <td>{{isNaN(project.done_tasks_count / project.tasks_count) ? 0 : Math.round((project.done_tasks_count / project.tasks_count) * 100) }} %</td>
                                <td>
                                    <a href="#" class="btn btn-primary" @click.prevent="editModal(project)">
                                        <i class="fa fa-edit"></i>
                                    </a>
                                    <a href="#" class="btn btn-danger" @click.prevent="deleteProject(project.id)">
                                        <i class="fa fa-trash"></i>
                                    </a>
                                    <router-link :to="{ name: 'projectTasks', params: { id: project.id } }" class="btn btn-info" >
                                        <i class="fa fa-tasks"></i>
                                    </router-link>
                                </td>
                            </tr>

                            </tbody></table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="projects" @pagination-change-page="getResults"></pagination>
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
                        <h5 v-if="editMode" class="modal-title" id="addNewModalLabel">Edit Project</h5>
                        <h5 v-else="editMode" class="modal-title" id="addNewModalLabel">Add New Project</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateProject() : createProject()">
                        <div class="modal-body">

                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name" placeholder="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
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
                projects: {},
                editMode: false,
                form: new Form({
                    id: '',
                    name: ''
                })
            }
        },
        methods: {
            newModal(){
                this.form.reset();
                this.editMode = false;
                $('#addNewModal').modal('show');
            },
            editModal(project){
                this.form.reset();
                this.editMode = true;
                $('#addNewModal').modal('show');
                this.form.fill(project);
            },
            getResults(page = 1) {
                axios.get('/project?page=' + page)
                    .then(response => {
                        this.projects = response.data;
                    });
            },
            loadProjects(){
                axios.get("/project").then(({data}) => {(this.projects = data)
                console.log(data)});
            },
            createProject(){
                this.form.post('/project').then(() => {
                    Fire.$emit('dataLoaded');
                    $('#addNewModal').modal('hide');
                    toast({
                        type: 'success',
                        title: 'Project created successfully'
                    });
                }).catch();

            },
            updateProject(){
                this.form.patch('/project/'+this.form.id)
                    .then(() => {
                        Fire.$emit('dataLoaded');
                        $('#addNewModal').modal('hide');
                        toast({
                            type: 'success',
                            title: 'Project Updared successfully'
                        });
                    })
                    .catch();
            }
            ,
            deleteProject(id){
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
                            this.form.delete('/project/' + id).then(() => {
                                Fire.$emit('dataLoaded');
                                Swal(
                                    'Deleted!',
                                    'The project has been deleted.',
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
            this.loadProjects();
            Fire.$on('dataLoaded', () => {
                this.loadProjects();
            });
        }
    }
</script>
