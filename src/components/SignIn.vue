<template>
  <v-container fluid>
    <v-layout row wrap>
      <v-form @submit.prevent="onFormSubmit" class="login-form auth-form">
        <v-flex xs6 offset-xs3 class="text-xs-center" mt-5>
          <h1>Аккаунт</h1>
        </v-flex>
        <v-flex sm6 offset-sm3 xs6 offset-xs3 mt-3>
          <form>
            <v-layout column>
              <v-flex>
                <v-text-field
                  v-model="login"
                  name="login"
                  label="Login"
                  id="login"
                  type="text"
                  color="light-blue accent-3"
                  required
                ></v-text-field>
              </v-flex>
              <v-flex>
                <v-text-field
                  name="password"
                  label="Password"
                  v-model="password"
                  id="password"
                  type="password"
                  color="light-blue accent-3"
                  required
                ></v-text-field>
              </v-flex>
              <v-flex class="text-xs-center" mt-5>
                <div class="enterBtn">
                  <v-btn
                    
                    color="light-blue accent-3"
                    type="submit"
                    >Войти</v-btn
                  >
                </div>
              </v-flex>
            </v-layout>
          </form>
        </v-flex>
      </v-form>
    </v-layout>
  </v-container>
</template>



<script>
import { doLogin } from "@/netClient/authService";
export default {
  // @click="onLoginClicked"
  name: "SignIn",
  data: () => ({
    login: '',
    password: '',
  }),
  components: {},
  async mounted() {},
  methods: {
    async onFormSubmit() {
      try {
        const token = await doLogin(this.login.trim(), this.password.trim());
        console.warn({ token });
        if (token) {
          this.$router.push("/");
        }
      } catch (error) {
        console.error({ error });
      }
    },
  },
};
</script>
