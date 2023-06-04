<template>
  <v-dialog
      v-model="isDialogShown"
      width="600px"
      persistent
  >
    <v-card>
      <v-card>
        <v-card-title>
          <span v-if="project">Edit project</span>
          <span v-else>Create project</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col
                  cols="12"
              >
                <v-text-field
                    v-model="title"
                    label="Title"
                    required
                    :error-messages="validationErrors['title']"
                ></v-text-field>
              </v-col>
              <v-col
                  cols="12"
              >
                <v-textarea
                    v-model="description"
                    name="input-3-1"
                    label="Description"
                    :error-messages="validationErrors['description']"
                ></v-textarea>
              </v-col>
              <v-col
                  cols="12"
                  sm="6"
                  md="4"
              >
                <v-menu
                    ref="dueDatePicker"
                    v-model="dueDatePicker"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    max-width="290px"
                    min-width="auto"
                >
                  <template v-slot:activator="{ on }">
                    <v-text-field
                        v-model="dueDate"
                        label="Due Date"
                        v-on="on"
                        :error-messages="validationErrors['dueDate']"
                        readonly
                    ></v-text-field>
                  </template>
                  <v-date-picker
                      v-model="dueDate"
                      no-title
                      @input="dueDatePicker = false"
                  ></v-date-picker>
                </v-menu>
              </v-col>
              <v-col
                  cols="12"
                  sm="6"
                  md="4"
              >
                <v-text-field
                    v-model="client"
                    label="Client"
                    required
                    :error-messages="validationErrors['client']"
                ></v-text-field>
              </v-col>
              <v-col
                  cols="12"
                  sm="6"
                  md="4"
              >
                <v-text-field
                    v-model="company"
                    label="Company"
                    required
                    :error-messages="validationErrors['company']"
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-alert v-if="validationError" type="error" dense>
                  {{ validationError }}
                </v-alert>
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
          <v-btn
              color="blue darken-1"
              text
              @click="submit"
              :loading="loading"
          >
            {{ submitBtnTitle }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from "axios";
import {CREATE_PROJECT, UPDATE_PROJECT} from "@/routes";

export default {
  name: 'ProjectDialog',
  props: {
    project: Object,
    isShown: Boolean,
  },
  data: () => ({
    valid: false,
    dueDatePicker: false,
    loading: false,
    title: '',
    description: '',
    dueDate: '',
    client: null,
    company: null,
    validationErrors: {},
    validationError: null,
  }),
  computed: {
    isDialogShown() {
        return this.isShown;
    },
    submitBtnTitle() {
      return this.project ? 'Save' : 'Create';
    },
  },
  watch: {
    isShown(val) {
      if (val) {
        this.init();
      }
    },
  },
  methods: {
    init() {
      this.reset();

      if (!this.project) {
        return;
      }

      this.title = this.project.title;
      this.description = this.project.description;
      this.dueDate = this.project.dueDate;

      if (this.project.client) {
        this.client = this.project.client;
      }

      if (this.project.company) {
        this.company = this.project.company;
      }
    },
    reset() {
      this.title = '';
      this.description = '';
      this.dueDate = '';
      this.client = null;
      this.company = null;
      this.validationErrors = {};
      this.validationError = null;

      this.loading = false;
    },
    close() {
      this.$emit('close');
    },
    getPayload() {
      return {
        title: this.title,
        description: this.description,
        dueDate: this.dueDate,
        client: this.client,
        company: this.company,
      };
    },
    async submit() {
      const payload = this.getPayload();
      this.loading = true;

      try {
        if (this.project) {
          await axios.put(UPDATE_PROJECT.replace('{id}', this.project.id), payload);
        } else {
          await axios.post(CREATE_PROJECT, payload);
        }
      } catch (error) {
        this.handleSubmitError(error);

        return;
      } finally {
        this.loading = false;
      }

      this.$emit('reload');
      this.close();
    },
    handleSubmitError(error) {
      if (error.response.status !== 422) {
        return;
      }

      this.validationError = null;
      this.validationErrors = {};
      error.response.data.violations.forEach((violation) => {
        if (violation.propertyPath === '') {
          this.validationError = violation.message;
          return;
        }

        this.validationErrors[violation.propertyPath] = violation.message;
      });
    },
  },
};
</script>
