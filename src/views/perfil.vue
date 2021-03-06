<template>
  <div>
    <v-alert dense text :type="alert.type" v-if="alert.show">
      {{ alert.message }}
    </v-alert>
    <v-row>
      <v-col xs="12" sm="12" md="12" lg="6" xl="6">
        <v-card class="pa-4">
          <v-row no-gutters>
            <v-col md="6" sm="12">
              <p class="subtitle-2">Nombre</p>
              <p class="body-1 grey--text">
                {{
                  `${datosPerfil.primer_nombre} ${datosPerfil.segundo_nombre}`
                }}
              </p>
            </v-col>
            <v-col md="6" sm="12">
              <p class="subtitle-2">Apellido</p>
              <p class="body-1 grey--text">
                {{
                  `${datosPerfil.primer_apellido} ${datosPerfil.segundo_apellido}`
                }}
              </p>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <p class="subtitle-2">Correo electrónico</p>
              <p class="body-1 grey--text">{{ datosPerfil.email }}</p>
            </v-col>
            <v-col>
              <p class="subtitle-2">Ultima conexion</p>
              <p class="body-1 grey--text">
                {{
                  datosPerfil.creacion
                    | moment("timezone", "America/El_Salvador", "LL")
                }}
              </p>
            </v-col>
          </v-row>
          <v-divider class="mt-4"></v-divider>
          <div class="mt-4">
            <p class="title">Actualizar email</p>
            <v-text-field
              prepend-icon="mdi-email"
              name="email"
              v-model="email"
              :rules="emailRules"
              @blur="validEmail()"
              label="Email "
              outlined
              dense
            >
            </v-text-field>
            <div class="text-center">
              <v-btn
                color="blueMinsal"
                rounded
                class="white--text"
                @click="cambiarEmail()"
              >
                <v-icon class="mr-1">mdi-content-save</v-icon>
                Guardar
              </v-btn>
            </div>
          </div>
        </v-card>
      </v-col>
      <v-col xs="12" sm="12" md="12" lg="6" xl="6">
        <v-form @submit.prevent="cambiarPassword" ref="passForm">
          <v-card class="py-4 px-8">
            <v-text-field
              prepend-icon="mdi-lock"
              name="password"
              v-model="password.value"
              label="Contraseña Actual"
              :type="password.show ? 'text' : 'password'"
              outlined
              dense
              :append-icon="
                password.show ? 'mdi-eye-off-outline' : 'mdi-eye-outline'
              "
              @click:append="password.show = !password.show"
              :error="password.state"
              :error-messages="password.message"
            >
            </v-text-field>
            <v-text-field
              prepend-icon="mdi-lock"
              name="newPassword"
              v-model="new_password.value"
              label="Nueva Contraseña"
              :type="new_password.show ? 'text' : 'password'"
              :append-icon="
                new_password.show ? 'mdi-eye-off-outline' : 'mdi-eye-outline'
              "
              :rules="newPasswordRules"
              outlined
              dense
              @click:append="new_password.show = !new_password.show"
              :error="new_password.state"
              :error-messages="new_password.message"
            >
            </v-text-field>
            <v-text-field
              prepend-icon="mdi-lock"
              name="confirmPassword"
              v-model="confirm_password.value"
              label="Confirmar Contraseña"
              :rules="confirmPasswordRules"
              :type="confirm_password.show ? 'text' : 'password'"
              outlined
              :append-icon="
                confirm_password.show
                  ? 'mdi-eye-off-outline'
                  : 'mdi-eye-outline'
              "
              dense
              @click:append="confirm_password.show = !confirm_password.show"
              :error="confirm_password.state"
              :error-messages="confirm_password.message"
            >
            </v-text-field>
            <v-card-actions class="flex justify-center">
              <v-btn
                color="blueMinsal"
                rounded
                class="white--text"
                type="submit"
              >
                <v-icon class="mr-1">mdi-content-save</v-icon>
                Guardar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-form>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { mapActions, mapMutations, mapState } from "vuex";
import jwtDecode from "jwt-decode";

export default {
  name: "perfil",
  data: () => ({
    password: {
      value: null,
      show: false,
      state: false,
      message: null,
    },
    new_password: {
      value: null,
      show: false,
      state: false,
      message: null,
    },
    confirm_password: {
      value: null,
      show: false,
      state: false,
      message: null,
    },
    email: null,
    emailRules: [
      (v) =>
        /^(?!(?:(?:\x22?\x5C[\x00-\x7E]\x22?)|(?:\x22?[^\x5C\x22]\x22?)){255,})(?!(?:(?:\x22?\x5C[\x00-\x7E]\x22?)|(?:\x22?[^\x5C\x22]\x22?)){65,}@)(?:(?:[\x21\x23-\x27\x2A\x2B\x2D\x2F-\x39\x3D\x3F\x5E-\x7E]+)|(?:\x22(?:[\x01-\x08\x0B\x0C\x0E-\x1F\x21\x23-\x5B\x5D-\x7F]|(?:\x5C[\x00-\x7F]))*\x22))(?:\.(?:(?:[\x21\x23-\x27\x2A\x2B\x2D\x2F-\x39\x3D\x3F\x5E-\x7E]+)|(?:\x22(?:[\x01-\x08\x0B\x0C\x0E-\x1F\x21\x23-\x5B\x5D-\x7F]|(?:\x5C[\x00-\x7F]))*\x22)))*@(?:(?:(?!.*[^.]{64,})(?:(?:(?:xn--)?[a-z0-9]+(?:-[a-z0-9]+)*\.){1,126}){1,}(?:(?:[a-z][a-z0-9]*)|(?:(?:xn--)[a-z0-9]+))(?:-[a-z0-9]+)*)|(?:\[(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){7})|(?:(?!(?:.*[a-f0-9][:\]]){7,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?)))|(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){5}:)|(?:(?!(?:.*[a-f0-9]:){5,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3}:)?)))?(?:(?:25[0-5])|(?:2[0-4][0-9])|(?:1[0-9]{2})|(?:[1-9]?[0-9]))(?:\.(?:(?:25[0-5])|(?:2[0-4][0-9])|(?:1[0-9]{2})|(?:[1-9]?[0-9]))){3}))\]))$/.test(
          v
        ) || "email no valido",
    ],
    alert: {
      show: false,
      type: "",
      message: "",
    },
  }),
  methods: {
    ...mapActions(["getUserDetail"]),
    ...mapMutations(["setUserInfo", "setToken"]),
    async cambiarEmail() {
      try {
        if (this.validEmail()) {
          await this.showLoader();
          const response = await this.http_client(
            "/api/change/email",
            {
              email: this.userDetail.email,
              new_email: this.email,
            },
            "post"
          );
          let token = response.data.token;
          let refresh = response.data.refresh_token;
          localStorage.setItem("token", token);
          localStorage.setItem("refresh_token", refresh);
          this.setToken(token);
          await this.setUserInfo(jwtDecode(token));
          await this.getUserDetail(this.email);
          this.temporalAlert({
            show: true,
            message: "Correo actualizado con exito",
            type: "success",
          });
        }
      } catch (e) {
        this.temporalAlert({
          show: true,
          message: e.response.data.status,
          type: "error",
        });
      } finally {
        await this.hideLoader();
      }
    },
    
    async cambiarPassword() {
      if (this.$refs.passForm.validate()) {
        try {
          this.showLoader();
          const response = await this.http_client(
            "/api/change/password",
            {
              old_password: this.password.value,
              new_password: this.new_password.value,
              repeat_password: this.confirm_password.value,
            },
            "post"
          );
          this.alert.type = "success";
          this.alert.show = true;
          this.alert.message = "Contraseña Actualizada";
          setTimeout(() => {
            this.alert.show = false;
          }, 5000);

          this.password.value = null;
          this.password.state = null;
          this.password.message = null;
          this.new_password.value = null;
          this.new_password.state = null;
          this.new_password.message = null;
          this.confirm_password.value = null;
          this.confirm_password.state = null;
          this.confirm_password.message = null;
        } catch (e) {
          this.temporalAlert({
            show: true,
            message: e.response.data.message
              ? e.response.data.message
              : "Ocurrió un error actualizando la contraseña",
            type: "error",
          });
        } finally {
          this.hideLoader();
          this.$refs.passForm.reset();
        }
      }
    },
    feelback(errors) {
      errors.forEach((error) => {
        if (error.propiedad === "new_password") {
          this.new_password.state = true;
          this.new_password.message = error.error;
        }
        if (error.propiedad === "repeat_password") {
          this.confirm_password.state = true;
          this.confirm_password.message = error.error;
        }
      });
    },
    validEmail() {
      return this.isEmail(this.email);
    },
  },
  computed: {
    ...mapState(["userDetail", "userInfo"]),
    newPasswordRules() {
      return [
        (v) => (v !== null && v !== "") || "Este campo es requerido",
        (v) =>
          this.isPassword(v) ||
          "La contraseña debe tener al menos 1 minuscula, 1 mayuscula, 1 caracter especial, 1 numero y minimo 8 caracteres",
      ];
    },
    confirmPasswordRules() {
      return [
        (v) => (v !== null && v !== "") || "Este campo es requerido",
        (v) =>
          this.isPassword(v) ||
          "La contraseña debe tener al menos 1 minuscula, 1 mayuscula, 1 caracter especial, 1 numero y minimo 8 caracteres",
        (v) => v == this.new_password.value || "Las contraseñas no coinciden",
      ];
    },
    datosPerfil() {
      const usuario_dummy = {
        email: "dummy@mail.com",
        fecha_creacion: "1999-10-10T11:15:25Z",
        id_persona: "",
        primer_apellido: "",
        primer_nombre: "",
        segundo_apellido: "",
        segundo_nombre: "",
        ultima_conexion: "1999-10-11T09:09:28Z",
      };
      let usuario;
      if (Object.keys(this.userDetail).includes("data")) {
        usuario = this.userDetail.data;
      } else {
        usuario = usuario_dummy;
      }
      return usuario;
    },
  },
};
</script>

<style scoped></style>
