<template>
  <div class="bg-white shadow rounded px-3 pt-3 pb-5 border border-white" title="Double click to start edit issue"
    v-on:dblclick="counter += 1, isEdit = true" v-if="!isEdit">
    <div class="flex justify-between">
      <p class="text-gray-700 font-semibold font-sans tracking-wide text-sm">{{ task.description }}</p>

      <img class="w-6 h-6 rounded-full ml-3" :src="`https://i.pravatar.cc/300?img=${task.id}`" alt="Avatar">
    </div>
    <div class="text-xs text-gray-700">
      <span>Assigned: <strong>{{ task.assignee }}</strong></span>
    </div>
    <div class="flex mt-4 justify-between items-center">
      <span class="text-sm text-gray-600">{{ currentDateTime(task.deadline_at) }}</span>
      <button class="text-sm text-red-700 gentle-hover-shake" title="Delete this task" @click="doDelete(task.id)"><i
          class="fa-solid fa-trash"></i></button>
      <!-- <badge v-if="task.is_done" :color="badgeColor">{{(task.is_done === true) ? 'Done' : 'To Do'}}</badge> -->
    </div>
  </div>
  <div class="bg-white shadow rounded px-3 pt-3 pb-5 border border-white" v-else>
    <form role="form" @submit.prevent="doSubmit(task.id)">
      <div class="flex justify-between">
        <input
          class="shadow appearance-none border rounded w-full px-1 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          id="issue" type="text" placeholder="Issue" v-model="form.description">
      </div>
      <div class="flex justify-between mt-1">
        <input
          class="shadow appearance-none border rounded w-full px-1 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          id="assignee" type="text" placeholder="Assignee" v-model="form.assignee">
      </div>
      <div class="flex justify-between mt-1">
        <input
          class="shadow appearance-none border rounded w-full px-1 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          id="deadline_at" type="date" placeholder="deadline_at" v-model="form.deadline_at">
      </div>
      <div class="flex mt-4 justify-between items-center">
        <button title="Cancel edit task"
          class="bg-gray-500 hover:bg-gray-700 text-white px-1 rounded focus:outline-none focus:shadow-outline"
          type="button" @click="cancel()">
          Cancel
        </button>
        <button title="Save this task"
          class="bg-blue-700 hover:bg-blue-500 text-white px-1 rounded focus:outline-none focus:shadow-outline"
          type="submit">
          Save
        </button>
      </div>
    </form>
  </div>
</template>
<script>
import Badge from "./Badge.vue";
import moment from 'moment';
import axios from 'axios';
export default {
  components: {
    Badge
  },
  props: {
    column: Object,
    task: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    badgeColor() {
      const mappings = {
        ToDo: "purple",
        Done: "blue",
      };
      return mappings[this.task.type] || mappings.default;
    }
  },
  data() {
    return {
      form: {
        assignee: this.task.assignee,
        description: this.task.description,
        deadline_at: moment(this.task.deadline_at).format('YYYY-MM-DD')
      },
      counter: 0,
      isEdit: false
    }
  },
  methods: {
    currentDateTime(val) {
      return moment(val).format('MMMM Do YYYY, h:mm:ss a')
    },
    doDelete(id) {
      this.$parent.$parent.doDelete(this.column.id, id)
    },
    cancel() {
      this.isEdit = false;
    },
    doSubmit(id) {
      try {
        let response = axios.put("http://localhost:3000/api/update/" + id, this.form, {
          headers: {
            'Content-Type': 'application/json'
          }
        }).then((res) => {
          this.task = res.data.data;
          this.cancel();
        });
            this.$swal.fire({
                position: 'center',
                icon: 'success',
                title: 'Task has been updated',
                showConfirmButton: false,
                timer: 1700
                });
      } catch (error) {
        console.log(error)
      }
    },
  }
};
</script>
