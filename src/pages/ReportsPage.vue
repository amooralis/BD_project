<template>
  <div class="all">
    <template>
      <div>
        <v-toolbar color="white">
          <v-toolbar-title>Все отчёты</v-toolbar-title>
          <v-divider class="mx-5" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            color="light-blue accent-3"
            hide-details
          ></v-text-field>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="light-blue accent-3" dark class="mb-2" v-on="on" @click="startCreateReport"
                >+ Создать новый отчёт</v-btn
              >
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm12 md12>
                      <v-autocomplete
                        v-model="editedItem.employee"
                        :items="this.employeesNames"
                        label="Работник*"
                        color="light-blue accent-3"
                        required
                      ></v-autocomplete>
                    </v-flex>

                    <v-flex xs12 sm8 md8>
                      <v-text-field
                        v-model="editedItem.dateFrom"
                        label="Дата начала отчёта*"
                        type="date"
                        color="light-blue accent-3"
                        required
                      ></v-text-field>
                    </v-flex>

                    <v-flex xs12 sm8 md8>
                      <v-text-field
                        v-model="editedItem.dateTo"
                        label="Дата окончания отчёта*"
                        type="date"
                        color="light-blue accent-3"
                        required
                      ></v-text-field>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="light-blue accent-3" @click="close"
                  >Отменить</v-btn
                >
                <v-btn color="green accent-2" @click="submitCreateReport()">Сохранить</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
        <v-data-table
          :headers="headers"
          :items="reports"
          :search="search"
          item-key="name"
          class="elevation-1"
        >

          <template v-slot:item.download="{ item }">
            <v-icon color="light-blue accent-3" @click="downloadItem(item)">
              download
            </v-icon>
          </template>
        </v-data-table>
      </div>
    </template>
  </div>
</template>

<script>
import { 
  getAllReports,
  createReport,
    } from "@/netClient/reportService";
import {getAllEmployees} from "@/netClient/employeesService";
export default {
  name: "ReportsPage",
  data: () => ({
    dialog: false,
    expand: [],
    employees:[],
    employeesNames: [],
    search: "",
    headers: [
      {
        text: "Работник",
        align: "left",
        sortable: false,
        value: "reportedEmployee.name",
      },
      { text: "От", value: "dateFrom" },
      { text: "До", value: "dateTo" },

      { text: "", value: "download", sortable: false },
    ],
    reports: [],
    reportsObject: [],
    currentReport: [],
    editedIndex: -1,
    editedItem: {
      employee: "",
      reportedEmployeeId: "",
      dateFrom: null,
      dateTo: null,
    },
    defaultItem: {
      employee: "",
      dateFrom: "-",
      dateTo: "-",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1
        ? "Создание нового отчёта"
        : "Редактировать";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
  },

  created() {
		this.refresh();
	},

  methods: {
    
    refresh() {
			this.fetchReportsList();
      this.fetchEmployeesList();
		},

    async fetchReportsList() {
      try {
        this.reports = await getAllReports();
      } catch (error){
        console.error({ error });
      }
    },

    async fetchEmployeesList() {
      try {
        this.employees = await getAllEmployees();
      } catch (error){
        console.error({ error });
      }
    },

    getEmployeesNames() {
      this.employees.forEach((element) => {
      this.employeesNames.push(element.name);  
      });
    },

    startCreateReport(){
      this.fetchEmployeesList();
      this.getEmployeesNames();
    },

    async submitCreateReport(){
      this.employees.forEach((element) => {
        if (element.name == this.editedItem.employee) {
          this.editedItem.reportedEmployeeId = element.id;
        }
      });
      if (this.editedItem.reportedEmployeeId) {
        try {
          await createReport(this.editedItem.reportedEmployeeId, this.editedItem.dateFrom, this.editedItem.dateTo);
        } catch (error) {
          console.error({ error });
        }

        this.dialog = false;
        this.refresh();
      }
    },

    async downloadItem(item) {
      alert("Скачать отчёт?");
      this.currentReport = item;
      try {
        window.open('http://localhost:3001/api/reports/download/' + this.currentReport.id + '?x-access-token='+localStorage.accessToken)
				this.refresh()
			} catch (error) {
				console.error({ error });
			}
    },

    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.reports[this.editedIndex], this.editedItem);
      } else {
        this.reports.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
