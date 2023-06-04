<template v-if="project">
  <v-dialog
      v-model="isDialogShown"
      width="600px"
      persistent
  >
    <v-card>
      <v-card>
        <v-card-title>
          <span>Task list for {{ project.title }}</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row v-if="tasks.length > 0">
              <v-col>
                <div v-for="task in tasks" v-bind:key="task.id">
                  <v-checkbox
                      class="mt-0 mb-0 d-inline-block"
                      v-model="completedTasks"
                      :label="task.description"
                      :value="task.id"
                      @change="toggleTask(task)"
                  ></v-checkbox>

                  <v-btn
                      icon
                      class="float-right"
                      @click="deleteTask(task)"
                  >
                    <v-icon title="Delete task">mdi-delete</v-icon>
                  </v-btn>
                </div>
              </v-col>
            </v-row>

            <v-row>
              <v-col>
                <v-text-field
                    v-model="newTaskDescription"
                    label="Description"
                ></v-text-field>

                <v-btn
                    color="primary"
                    block
                    :disabled="!canCreate"
                    @click="createTask"
                    :loading="isSubmitting"
                >
                  Add task
                </v-btn>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
              color="blue darken-1"
              text
              @click="close"
          >
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from "axios";
import {CREATE_TASK, DELETE_TASK, GET_TASKS_FOR_PROJECT, UPDATE_TASK} from "@/routes";

export default {
  name: 'TaskListDialog',
  props: {
    project: Object,
    isShown: Boolean,
  },
  data: () => ({
    tasks: [],
    completedTasks: [],
    newTaskDescription: '',
    isSubmitting: false,
  }),
  computed: {
    isDialogShown() {
      return this.isShown;
    },
    canCreate() {
      return this.newTaskDescription.trim().length > 0;
    }
  },
  methods: {
    close() {
      this.$emit('close');
      this.$emit('reload');
    },
    async fetchTasksList() {
      const url = process.env.VUE_APP_API_BASE_URL + GET_TASKS_FOR_PROJECT.replace('{id}', this.project.id);
      const response = await axios.get(url);

      this.tasks = response.data['hydra:member'];
      this.tasks.forEach((task) => {
        if (task.isCompleted) {
          this.completedTasks.push(task.id);
        }
      })
    },
    async toggleTask(task) {
      const url = process.env.VUE_APP_API_BASE_URL + UPDATE_TASK.replace('{id}', task.id);
      const isCompleted = this.completedTasks.includes(task.id);

      await axios.patch(url, {isCompleted: isCompleted}, {headers: {'Content-Type': 'application/merge-patch+json'}});
    },
    async deleteTask(task) {
      const url = process.env.VUE_APP_API_BASE_URL + DELETE_TASK.replace('{id}', task.id);

      await axios.delete(url);
      await this.fetchTasksList();
    },
    async createTask() {
      const url = process.env.VUE_APP_API_BASE_URL + CREATE_TASK;

      this.isSubmitting = true;

      await axios.post(url, {
        description: this.newTaskDescription,
        project: this.project['@id'],
        isCompleted: false,
      });
      await this.fetchTasksList();

      this.newTaskDescription = '';
      this.isSubmitting = false;
    },
  },
  mounted() {
    this.fetchTasksList();
  }
}
</script>
