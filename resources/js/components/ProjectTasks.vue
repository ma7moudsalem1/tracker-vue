<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title"><a href="#" @click.prevent="infoModal">{{ project.name }}</a> Tasks List [ {{ project.tasks_count }} ]</h3>

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
                                <th>Status</th>
                                <th>Action</th>
                            </tr>
                            <tr v-for="task in tasks.data" :key="task.id">
                                <td>{{task.id}}</td>
                                <td>{{task.name}}</td>
                                <td>{{ task.status }}</td>
                                <td>
                                    <a href="#" class="btn btn-primary" @click.prevent="editModal(task)">
                                        <i class="fa fa-edit"></i>
                                    </a>
                                    <a href="#" class="btn btn-danger" @click.prevent="deleteTask(task.id)">
                                        <i class="fa fa-trash"></i>
                                    </a>
                                </td>
                            </tr>

                            </tbody></table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="tasks" @pagination-change-page="getResults"></pagination>
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
                        <h5 v-if="editMode" class="modal-title" id="addNewModalLabel">Edit Task</h5>
                        <h5 v-else="editMode" class="modal-title" id="addNewModalLabel">Add New Task</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateTask() : createTask()">
                        <div class="modal-body">

                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name" placeholder="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <input type="hidden" name="project_id" v-model="form.project_id" />

                            <div class="form-group">
                                <select v-model="form.status" name="status"
                                        class="form-control" :class="{ 'is-invalid': form.errors.has('status') }" id="status">
                                    <option>Select Status</option>
                                    <option value="In Progress">In Progress</option>
                                    <option value="done">Done</option>
                                </select>
                                <has-error :form="form" field="status"></has-error>
                            </div>

                            <div class="form-group">
                                <textarea  v-model="form.comment" name="comment" placeholder="Short Comment (Optional)"
                                           class="form-control" :class="{ 'is-invalid': form.errors.has('comment') }" id="comment"
                                           cols="30" rows="3"></textarea>

                                <has-error :form="form" field="comment"></has-error>
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


        <!-- Modal -->
        <div class="modal fade" id="infoModal" tabindex="-1" role="dialog" aria-labelledby="infoModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="infoModalLabel">Project info</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                        <div class="modal-body">

                           <p>Project Name: {{ project.name }}</p>
                           <p>Created At: {{ project.created_at }}</p>
                           <p>Comment: {{ project.comment }}</p>
                           <p>Last Update: {{ project.updated_at }}</p>
                           <p>Total Tasks: {{ project.tasks_count }}</p>
                           <p>Total Tasks Done: {{ project.done_tasks_count }}</p>
                           <p>Total Status: {{isNaN(project.done_tasks_count / project.tasks_count) ? 0 : Math.round((project.done_tasks_count / project.tasks_count) * 100) }} %</p>



                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        data(){
            return {
                tasks: {},
                project: {},
                editMode: false,
                form: new Form({
                    id: '',
                    name: '',
                    project_id: '',
                    comment: '',
                    status: '',
                })
            }
        },
        methods: {
            newModal(){
                this.form.reset();
                this.form.project_id = this.$route.params.id;
                this.editMode = false;
                $('#addNewModal').modal('show');
            },
            infoModal(){
                $('#infoModal').modal('show');
            },
            editModal(task){
                this.form.reset();
                this.editMode = true;
                $('#addNewModal').modal('show');
                this.form.fill(task);
            },
            getResults(page = 1) {
                axios.get(this.form.project_id+'/tasks/get?page=' + page)
                    .then(response => {
                        this.tasks = response.data;
                    });
            },
            loadTasks(){
                axios.get(this.form.project_id+'/tasks/get').then(({data}) => (this.tasks = data));
            },
            loadProject(id){
                axios.get('/project/'+ id).then(({data}) => (this.project = data));
            },
            createTask(){
                this.form.post('/task').then(() => {
                    Fire.$emit('dataLoaded');
                    $('#addNewModal').modal('hide');
                    toast({
                        type: 'success',
                        title: 'Task created successfully'
                    });
                }).catch();

            },
            updateTask(){
                this.form.patch('/task/'+this.form.id)
                    .then(() => {
                        Fire.$emit('dataLoaded');
                        $('#addNewModal').modal('hide');
                        toast({
                            type: 'success',
                            title: 'Task Updated successfully'
                        });
                    })
                    .catch();
            }
            ,
            deleteTask(id){
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
                            this.form.delete('/task/' + id).then(() => {
                                Fire.$emit('dataLoaded');
                                Swal(
                                    'Deleted!',
                                    'This task has been deleted.',
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
            this.form.project_id = this.$route.params.id;
            this.loadProject(this.form.project_id);
            this.loadTasks();

            Fire.$on('dataLoaded', () => {
                this.loadTasks();
                this.loadProject(this.form.project_id);
            });
        }
    }
</script>
