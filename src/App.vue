<template>
    <div id="app">
        <div class="flex justify-center">
            <div class="min-h-screen flex overflow-x-scroll py-12">
                <div class="rounded-lg px-3 py-3 column-width rounded mr-4">
                    <form class="bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4" role="form"
                        @submit.prevent="doSubmit()">
                    <h1 class="text-gray-700 font-semibold font-sans tracking-wide text-center">Create New Task</h1>
                    <hr>
                        <div class="mb-4 mt-4">
                            <label class="block text-gray-700 text-sm font-bold mb-2" for="assignee">
                                Assignee
                            </label>
                            <input
                                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                                id="assignee" type="text" placeholder="Assignee" v-model="form.assignee" required>
                        </div>
                        <div class="mb-4">
                            <label class="block text-gray-700 text-sm font-bold mb-2" for="task">
                                Task
                            </label>
                            <input
                                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                                id="task" type="text" placeholder="Task" v-model="form.description" required>
                        </div>
                        <div class="mb-4">
                            <label class="block text-gray-700 text-sm font-bold mb-2" for="deadline">
                                Deadline
                            </label>
                            <input
                                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                                id="deadline" type="date" placeholder="Deadline" v-model="form.deadline_at" required>
                        </div>
                        <div class="flex items-center justify-center">
                            <button
                                class="bg-blue-700 hover:bg-blue-500 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
                                type="submit">
                                Save
                            </button>
                        </div>
                    </form>
                </div>
                <div v-for="column in columns" :key="column.id"
                    class="bg-gray-100 rounded-lg px-3 py-3 column-width rounded mr-4">
                    <p class="text-gray-700 font-semibold font-sans tracking-wide text-sm">{{ column.title }}</p>
                    <!-- Draggable component comes from vuedraggable. It provides drag & drop functionality -->
                    <draggable :list="column.tasks" :animation="200" ghost-class="ghost-card" group="tasks"
                        @change="finish($event, column)">
                        <!-- Each element from here will be draggable and animated. Note :key is very important here to be unique both for draggable and animations to be smooth & consistent. -->
                        <task-card v-for="(task) in column.tasks" :key="task.id" :task="task" :column="column" class="mt-3 cursor-move">
                        </task-card>
                        <!-- </transition-group> -->
                    </draggable>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import draggable from "vuedraggable";
import TaskCard from "./components/TaskCard.vue";
import axios from 'axios';
export default {
    name: "App",
    components: {
        TaskCard,
        draggable
    },
    mounted() {
        this.getData();
    },
    data() {
        return {
            columns: Array,
            form: {
                assignee: '',
                description: '',
                deadline_at: ''
            }
        };
    },
    methods: {
        finish(evt, list) {
            if (evt.added) {
                try {
                    let response = axios.put("http://localhost:3000/api/update/progress/" + evt.added.element.id, {
                        is_done: list.is_done
                    });
                    const Toast = this.$swal.mixin({
                    toast: true,
                    position: 'top-end',
                    showConfirmButton: false,
                    timer: 3000,
                    timerProgressBar: true,
                    didOpen: (toast) => {
                        toast.addEventListener('mouseenter', this.$swal.stopTimer)
                        toast.addEventListener('mouseleave', this.$swal.resumeTimer)
                    }
                    })

                    Toast.fire({
                    icon: 'success',
                    title: 'Task moved successfully'
                    })
                } catch (error) {
                    console.log(error)
                }
                // console.log(list)
                // console.log(evt.added.element)
            }
        },
        getData() {
            var todos = []
            var dones = []
            axios
                .get("http://localhost:3000/api/list")
                .then((res) => {
                    if (res.data.data.todo) {
                        todos = res.data.data.todo;
                    }
                    if (res.data.data.done) {
                        dones = res.data.data.done;
                    }
                    this.columns = [
                        {
                            id: 0,
                            is_done: false,
                            title: "Task List",
                            tasks: todos
                        },
                        {
                            id: 1,
                            is_done: true,
                            title: "Done",
                            tasks: dones
                        }
                    ]
                });
        },
        doSubmit() {
            try {
                let response = axios.post("http://localhost:3000/api/store", this.form,{
                                    headers: {
                                    'Content-Type': 'application/json'
                                    }
                                }).then((res) => {
                                    // console.log(res.data.data)
                                    this.columns[0].tasks.push(res.data.data)
                                });
                this.form.assignee = ''
                this.form.description = ''
                this.form.deadline_at = ''
            this.$swal.fire({
                position: 'center',
                icon: 'success',
                title: 'New task has been added',
                showConfirmButton: false,
                timer: 1700
                });
            } catch (error) {
                console.log(error)
            }
        },
        doDelete(column,id) {
            let obj = this.columns[column].tasks.find(o => o.id === id);
            let idx = this.columns[column].tasks.indexOf(obj);
            this.columns[column].tasks.splice(idx,1)
            try {
                let response = axios.delete("http://localhost:3000/api/" + id);
            this.$swal.fire({
                position: 'center',
                icon: 'success',
                title: 'Task has been deleted',
                showConfirmButton: false,
                timer: 1700
                });
            } catch (error) {
                console.log(error)
            }
        }
    }
};
</script>

<style scoped>
.column-width {
    min-width: 320px;
    width: 320px;
}

/* Unfortunately @apply cannot be setup in codesandbox, 
but you'd use "@apply border opacity-50 border-blue-500 bg-gray-200" here */
.ghost-card {
    opacity: 0.5;
    background: #F7FAFC;
    border: 1px solid #4299e1;
}
</style>
