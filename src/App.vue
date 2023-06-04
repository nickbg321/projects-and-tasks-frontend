<template>
  <v-app style="background: #FAFAFA">
    <v-main>
      <v-container>
        <div class="d-flex justify-space-between mt-5 mb-5">
          <div class="text-h4 float-left">Projects and Tasks</div>

            <v-btn
                @click="openProjectModal"
                color="primary"
                large
            >
              New Project
            </v-btn>
        </div>

        <div>
          <v-data-table
              :headers="headers"
              :items="projects"
              :items-per-page="20"
              :loading="loading"
              :footer-props="footerProps"
              :server-items-length="total"
              :options.sync="options"
              class="elevation-2"
              @update:options="fetchProjects"
          >
            <template v-slot:[`item.status`]="{ item }">
              <v-chip
                  :color="getStatusColor(item.status)"
                  dark
              >
                {{ getHumanStatus(item.status) }}
              </v-chip>
            </template>

            <template v-slot:[`item.action`]="{ item }">
              <v-icon
                  @click="openTasksList(item)"
                  class="mr-2"
                  title="Manage tasks"
              >
                mdi-format-list-checkbox
              </v-icon>
              <v-icon
                  @click="editProject(item)"
                  class="mr-2"
                  title="Edit project"
              >
                mdi-pencil
              </v-icon>
              <v-icon
                  @click="deleteProject(item)"
                  title="Delete project"
              >
                mdi-delete
              </v-icon>
            </template>
          </v-data-table>

          <TasksListDialog
              v-if="selectedProject"
              :project="selectedProject"
              :is-shown="isTasksListDialogShown"
              @close="closeTasksListModal"
              @reload="fetchProjects"
          />

          <ProjectDialog
              :project="selectedProject"
              :is-shown="isProjectDialogShown"
              @close="closeProjectModal"
              @reload="fetchProjects"
          />
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import axios from "axios";
import {DELETE_PROJECT, GET_PROJECTS_COLLECTION} from "@/routes";
import ProjectDialog from "@/components/ProjectDialog.vue";
import TasksListDialog from "@/components/TasksListDialog.vue";

export default {
  name: 'App',
  components: {
    ProjectDialog,
    TasksListDialog,
  },
  data: () => ({
    headers: [
      { text: 'Title', value: 'title', sortable: false },
      { text: 'Description', value: 'description', sortable: false },
      { text: 'Due Date', value: 'dueDate', sortable: false },
      { text: 'Status', value: 'status', sortable: false },
      { text: 'Client', value: 'client', sortable: false },
      { text: 'Company', value: 'company', sortable: false },
      { text: 'Actions', value: 'action', sortable: false },
    ],
    footerProps: {
      'items-per-page-options': [10, 20, 50, 100],
    },
    options: {},
    total: 0,
    projects: [],
    loading: false,
    isProjectDialogShown: false,
    isTasksListDialogShown: false,
    selectedProject: null,
  }),
  methods: {
    getStatusColor(status) {
      switch (status) {
        case 'new':
          return '#2196F3';
        case 'pending':
          return '#FFC107';
        case 'done':
          return '#4CAF50';
         case 'failed':
          return '#F44336';
      }
    },
    getHumanStatus(status) {
      return status.charAt(0).toUpperCase() + status.slice(1);
    },
    async fetchProjects() {
      this.loading = true;

      const url = GET_PROJECTS_COLLECTION + '?itemsPerPage=' + this.options.itemsPerPage + '&page=' + this.options.page;
      const response = await axios.get(url);

      this.projects = response.data['hydra:member'];
      this.total = response.data['hydra:totalItems'];
      this.loading = false;
    },
    openProjectModal() {
      this.isProjectDialogShown = true;
    },
    closeProjectModal() {
      this.isProjectDialogShown = false;
      this.selectedProject = null;
    },
    closeTasksListModal() {
      this.isTasksListDialogShown = false;
      this.selectedProject = null;
    },
    openTasksList(project) {
      this.selectedProject = project;
      this.isTasksListDialogShown = true;
    },
    async deleteProject(project) {
      this.loading = true;

      await axios.delete(DELETE_PROJECT.replace('{id}', project.id));
      await this.fetchProjects();
    },
    async editProject(project) {
      this.selectedProject = project;
      this.isProjectDialogShown = true;
    },
  },
};
</script>
