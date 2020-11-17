<template>
  <v-layout row wrap>
    <v-flex lg12>
      <!--Change Password-->
      <app-card :fill-height="false">
        <div class="app-card-header">
          <div class="heading-block">
            <h4 class="heading">Change Password</h4>
          </div>
        </div>
        <div class="card-body">
          <v-form
            class="change-password-field"
            ref="passwordForm"
            lazy-validation
            v-model="passwordValid"
            @submit.prevent="validatePassword">
            <v-layout row wrap>
              <v-flex lg3 pt-0 pb-0>
                <v-text-field
                  flat
                  :v-ripple="false"
                  v-model="currentPassword"
                  label="Current Password"
                  placeholder="Enter Current Password"
                  class="i-v-text-field-default w-100"
                  type="password"
                  name="current-password"
                  :rules="currentPasswordRules">
                </v-text-field>
              </v-flex>
            </v-layout>
            <v-layout row wrap>
              <v-flex lg3 pt-0>
                <v-text-field
                  flat
                  :v-ripple="false"
                  v-model="newPassword"
                  label="New Password"
                  placeholder="Enter New Password"
                  class="i-v-text-field-default w-100 pt-0"
                  type="password"
                  name="new-password"
                  required
                  :rules="passwordRules">
                </v-text-field>
              </v-flex>
              <v-flex lg3 pt-0>
                <v-text-field
                  flat
                  :v-ripple="false"
                  v-model="repeatPassword"
                  label="Repeat Password"
                  placeholder="Repeat New Password"
                  class="i-v-text-field-default w-100 pt-0"
                  type="password"
                  name="repeat-password"
                  :rules="repeatPasswordRules">
                </v-text-field>
              </v-flex>
              <v-flex lg3 class="d-box flex-none align-center">
              <i-btn sm type="submit" color="primary" :disabled="passwordRequestSent" class="mb-2">Change Password</i-btn>
              </v-flex>
            </v-layout>
          </v-form>
        </div>
      </app-card>

      <!--Change Email-->
      <app-card :fill-height="false" class="mt-4">
        <div class="app-card-header">
          <div class="heading-block">
            <h4 class="heading">Change Email</h4>
          </div>
        </div>
        <div class="card-body">
          <v-form
            class="change-password-field"
            ref="emailForm"
            lazy-validation
            v-model="emailValid"
            @submit.prevent="validateEmail">
            <v-layout>
              <v-flex lg3 pt-0>
                <v-text-field
                  ref="emailUserRef"
                  flat
                  :v-ripple="false"
                  :placeholder="getEmail.user"
                  label="Email Adress"
                  class="i-v-text-field-default w-100"
                  type="email"
                  name="current-password"
                  :rules="emailRules">
                </v-text-field>
              </v-flex>
              <v-flex lg2 pt-0>
                <v-text-field
                  ref="emailDomainRef"
                  flat
                  :v-ripple="false"
                  :placeholder="getEmail.domain"
                  label="Email domain"
                  class="i-v-text-field-default i-v-domain-field w-100"
                  type="email"
                  name="new-password"
                  :rules="emailDomainRules">
                </v-text-field>
              </v-flex>
              <v-flex lg3 class="d-box flex-none align-center">
                <i-btn sm type="submit" color="primary" class="mt-2">Change Email</i-btn>
              </v-flex>
            </v-layout>
          </v-form>
        </div>
      </app-card>

      <!--Success Toaster-->
      <notifications group="success" position="bottom right" :max="toaster.max" width="350">
        <template slot="body" slot-scope="props">
          <div class="default-notification save-notification">
            <a class="close" @click="props.close">
              <v-icon>close</v-icon>
            </a>
            <a class="title">
              <img src="../assets/img/icons/success.svg" alt="">
              {{ props.item.title }}
            </a>
            <p v-html="props.item.text"></p>
          </div>
        </template>
      </notifications>

      <!-- Error Toaster-->
      <notifications group="error" position="bottom right" :max="toaster.max" width="350">
        <template slot="body" slot-scope="props">
          <div class="default-notification save-notification">
            <a class="close" @click="props.close">
              <v-icon>close</v-icon>
            </a>
            <a class="title danger">
              <img src="../assets/img/icons/cancel.svg" alt="">
              {{ props.item.title }}
            </a>
            <p v-html="props.item.text"></p>
          </div>
        </template>
      </notifications>
    </v-flex>
  </v-layout>
</template>

<script>
import axios from 'axios'
import {AppUrl} from '../assets/js/variables'
export default {
  name: 'ProfileChangePassword',
  data () {
    return {
      currentPassword: null,
      newPassword: null,
      repeatPassword: null,
      passwordValid: false,
      passwordRequestSent: false,
      emailValid: false,
      currentPasswordRules: [
        v => !!v || 'Field is required'
      ],
      passwordRules: [
        v => !!v || 'Field is required',
        v => (v && v.length >= 7) || 'Password must contain at least 8 characters',
        v => /[A-Z]/.test(v) || 'The password must contain uppercase letters',
        v => /[ !@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]/.test(v) || 'The password must contain special characters'
      ],
      repeatPasswordRules: [
        v => (v && v === this.newPassword) || 'Fields doesn\'t match'
      ],
      emailRules: [
        v => !!v || 'Field is required'
      ],
      emailDomainRules: [
        v => !!v || 'Field is required',
        v => /[.]/.test(v) || 'Please write valid domain'
      ],
      toaster: {
        max: 3
      }
    }
  },
  mounted () {
    if (!this.getEmail.user || !this.getEmail.domain) {
      this.getEmailRequest()
    }
  },
  methods: {
    validatePassword () {
      if (this.$refs.passwordForm.validate()) {
        this.snackbar = true
        this.passwordRequestSent = true
        this.changePasswordRequest().then((res) => {
          this.passwordRequestSent = false
          if (res === 1) {
            this.$refs.passwordForm.reset()
            this.$notify({
              group: 'success',
              title: 'Password Changed',
              text: 'New password Saved !',
              duration: 500,
              speed: 600
            })
          } else {
            this.$notify({
              group: 'error',
              title: 'Current Password is incorrect !',
              text: 'Please check the Current Password',
              duration: 500,
              speed: 600
            })
          }
        })
      }
    },
    changePasswordRequest () {
      return new Promise((resolve, reject) => {
        axios.post(AppUrl, {
          task: 'changePassword',
          payload: {
            current_password: this.currentPassword,
            new_password: this.newPassword,
            retype_password: this.repeatPassword
          }
        })
          .then(response => {
            if (response.data.payload) {
              resolve(1)
            } else {
              resolve(0)
            }
          })
          .catch(error => {
            console.log(error)
          })
      })
    },
    getEmailRequest () {
      this.$store.dispatch('profileSettingsModule/getEmail')
    },
    changeEmailRequest () {
      this.$store.dispatch('profileSettingsModule/changeEmail', { userPart: this.$refs.emailUserRef.internalValue, domainPart: this.$refs.emailDomainRef.internalValue }).then((res) => {
        this.getEmailRequest()
        if (res === 1) {
          this.$refs.emailForm.reset()
          this.$notify({
            group: 'success',
            title: 'Email Changed',
            text: 'New email Saved !',
            duration: 500,
            speed: 600
          })
        } else {
          this.$notify({
            group: 'error',
            title: 'Something went wrong',
            text: 'Please check the fileds',
            duration: 500,
            speed: 600
          })
        }
      })
    },
    commitEmailUser () {
      this.$store.commit('profileSettingsModule/COMMIT_emailUser', this.$refs.emailUserRef.internalValue)
    },
    commitEmailDomain () {
      this.$store.commit('profileSettingsModule/COMMIT_emailDomain', this.$refs.emailDomainRef.internalValue)
    },
    validateEmail () {
      if (this.$refs.emailForm.validate()) {
        this.snackbar = true
        this.commitEmailUser()
        this.commitEmailDomain()
        this.changeEmailRequest()
      }
    }
  },
  computed: {
    getEmail () {
      return this.$store.state.profileSettingsModule.email
    }
  }
}
</script>

<style scoped>

</style>
