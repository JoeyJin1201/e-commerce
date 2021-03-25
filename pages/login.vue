<template>
  <v-form
    ref="form"
    v-model="valid"
    lazy-validation
  >
    <v-text-field
      v-model="user.username"
      :rules="emailRules"
      label="E-mail"
      required
    />

    <v-text-field
      v-model="user.password"
      :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
      :type="show1 ? 'text' : 'password'"
      name="input-10-1"
      label="Normal with hint text"
      hint="At least 8 characters"
      counter
      required
      @click:append="show1 = !show1"
    />

    <v-btn
      :disabled="!valid"
      color="success"
      class="mr-4"
      @click="signin"
    >
      Log in
    </v-btn>

    <v-btn
      :disabled="!valid"
      color="success"
      class="mr-4"
      @click="validate"
    >
      Validate
    </v-btn>

    <v-btn
      color="error"
      class="mr-4"
      @click="reset"
    >
      Reset Form
    </v-btn>

    <v-btn
      color="warning"
      @click="resetValidation"
    >
      Reset Validation
    </v-btn>
    </v-btn>
  </v-form>
</template>

<script>
export default {
  data: () => ({
    user: {
      username: '',
      password: ''
    },
    status: {
      signIn: false
    },
    valid: true,
    name: '',
    nameRules: [
      v => !!v || 'Name is required',
      v => (v && v.length <= 10) || 'Name must be less than 10 characters'
    ],
    email: '',
    emailRules: [
      v => !!v || 'E-mail is required',
      v => /.+@.+\..+/.test(v) || 'E-mail must be valid'
    ],
    select: null,
    items: [
      'Item 1',
      'Item 2',
      'Item 3',
      'Item 4'
    ],
    checkbox: false,
    show1: false,
    password: 'Password',
    rules: {
      required: value => !!value || 'Required.',
      min: v => v.length >= 8 || 'Min 8 characters',
      emailMatch: () => ('The email and password you entered don\'t match')
    }
  }),

  methods: {
    validate () {
      this.$refs.form.validate()
    },
    reset () {
      this.$refs.form.reset()
    },
    resetValidation () {
      this.$refs.form.resetValidation()
    },
    signin () {
      const api = `${process.env.APIPATH}/admin/signin`
      const vm = this
      vm.status.signIn = true
      vm.$axios.post(api, vm.user).then((response) => {
        vm.status.signIn = false
        if (response.data.success) {
          const token = response.data.token
          const expired = response.data.expired
          document.cookie = `hexToken=${token};expires=${new Date(expired)};`
          vm.$router.push('/admin/dashboard/products')
          const newToken = document.cookie.replace(
            /(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/,
            '$1'
          )
          this.$axios.defaults.headers.common.Authorization = `${newToken}`
        }
      })
    }
  }
}
</script>
