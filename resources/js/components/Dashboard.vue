<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-lg-3 col-6">
                <!-- small box -->
                <div class="small-box bg-info">
                    <div class="inner">
                        <h3>{{ stats.projects }}</h3>

                        <p>Projects</p>
                    </div>
                    <div class="icon">
                        <i class="fas fa-project-diagram"></i>
                    </div>
                    <router-link to="/projects" class="small-box-footer">More info <i class="fa fa-arrow-circle-right"></i></router-link>
                </div>
            </div>
            <!-- ./col -->
            <div class="col-lg-3 col-6">
                <!-- small box -->
                <div class="small-box bg-success">
                    <div class="inner">
                        <h3>{{ stats.tasks }}</h3>

                        <p>Tasks</p>
                    </div>
                    <div class="icon">
                        <i class="fas fa-tasks"></i>
                    </div>
                    <router-link to="/tasks" class="small-box-footer">More info <i class="fa fa-arrow-circle-right"></i></router-link>
                </div>
            </div>
            <!-- ./col -->
            <div class="col-lg-3 col-6">
                <!-- small box -->
                <div class="small-box bg-danger">
                    <div class="inner">
                        <h3>{{ stats.done_task }}</h3>

                        <p>Done Tasks</p>
                    </div>
                    <div class="icon">
                        <i class="fa fa-check"></i>
                    </div>
                    <router-link to="/tasks" class="small-box-footer">More info <i class="fa fa-arrow-circle-right"></i></router-link>
                </div>
            </div>
            <!-- ./col -->
            <div class="col-lg-3 col-6">
                <!-- small box -->
                <div class="small-box bg-warning">
                    <div class="inner">
                        <h3>{{ stats.users }}</h3>

                        <p>Users</p>
                    </div>
                    <div class="icon">
                        <i class="fas fa-users"></i>
                    </div>
                    <router-link to="/users" class="small-box-footer">More info <i class="fa fa-arrow-circle-right"></i></router-link>
                </div>
            </div>
            <!-- ./col -->

        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                stats: [],
            }
        },
        methods: {
            loadStats(){
                axios.get("dashboard").then(({data}) => (this.stats = data));
            }
        }
        ,
        created() {
            this.loadStats();
        }
        ,
        mounted() {
            console.log('Component mounted.')
        }
    }
</script>
