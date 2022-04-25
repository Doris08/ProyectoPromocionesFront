<template>
  <v-app>

    <app-header></app-header>

    <app-aside-bar
      v-if="!$vuetify.breakpoint.xs && !$vuetify.breakpoint.sm"
    ></app-aside-bar>
    <v-main class="grey lighten-3">
      <div class="pa-4">
        <router-view :key="$route.path" />
      </div>
    </v-main>
    <app-bottom-navigation
      v-if="$vuetify.breakpoint.xs || $vuetify.breakpoint.sm"
    ></app-bottom-navigation>

     <v-container class="pa-0 pa-sm-2 justify-center d-flex" fill-height fluid>
    <!--      <v-layout align-center justify-center>-->
    <v-flex xs12 sm11 md10 lg9>
       <v-card rounded="lg">
        <v-card-title class="mb-3">Listado de avisos</v-card-title>
        <v-card-text>
            <v-flex text-xs-right>
                <v-btn
                        color="primary"
                        dark
                        class="mb-2"
                        v-bind="attrs"
                        v-on="on"
                    >
                        Nuevo Aviso
                </v-btn>
            </v-flex>
         <template>
            <v-card>
              <v-card-title>
                Avisos
                <v-spacer></v-spacer>
                <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="Search"
                  single-line
                  hide-details
                ></v-text-field>
              </v-card-title>
              <v-data-table
                :headers="headers"
                :items="avisos"
                :search="search"
              >
              </v-data-table>
            </v-card>
            </template>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-container>
  </v-app>
</template>


<script>
import AppHeader from "../components/AppHeader";
import AppAsideBar from "../components/AppAsideBar";
import AppBottomNavigation from "../components/AppBottomNavigation.vue";

export default {
  name: "layout",
  components: {
    AppHeader,
    AppAsideBar,
    AppBottomNavigation,
  },
  data: () => ({
    avisos: [],
    search: '',
    headers: [
      {
        text: "Nombre",
        align: "start",
        sortable: false,
        value: "nombre",
      },
      {
        text: "Descripcion",
        align: "start",
        sortable: false,
        value: "descripcion",
      },
      { text: "Estado", value: "activo", sortable: false, width: "100" },
      { text: 'Aciones', value: 'actions', sortable: false },
    ],

  }),

  methods: {
    async getAvisos() {
      try {
        const response = await this.http_client("/api/v1/promoavisos");
        this.avisos = response.data;
      } catch (e) {
        this.temporalAlert({
          show: true,
          message: e.response.data.message,
          type: "warning",
        });
      }
    },
    
  },
   async mounted() {
    await this.getAvisos();
  },
  
};
</script>


