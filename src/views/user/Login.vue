<template>
  <v-layout row justify-center>
		<v-dialog v-model="dialog" persistent max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">登录</span>
        </v-card-title>
        <v-card-text>
          <v-form v-model="valid" ref="form" lazy-validation>
            <v-text-field
              label="Username"
              v-model="username"
              :rules="usernameRules"
              :counter="10"
              required
            ></v-text-field>
            <v-text-field
              label="Password"
              v-model="password"
              :rules="passwordRules"
              type="password"
              required
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" flat @click.native="cancel">取消</v-btn>
          <v-btn color="blue darken-1" flat @click.native="submitLogin">登录</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import authApi from '../../api/user'
import {mapActions} from 'vuex'

export default {
  data: () => ({
    dialog: true,
    valid: true,
    username: '',
    usernameRules: [
      v => !!v || 'Name is required',
      v => v.length <= 15 || 'Name must be less than 15 characters'
    ],
    password: '',
    passwordRules: [
      v => !!v || 'Password is required',
      v => v.length > 5 || 'Password must be more then 5 characters'
      // v => /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'E-mail must be valid'
    ]
  }),
  methods: {
    ...mapActions([
      'login'
    ]),
    submitLogin () {
      if (!this.$refs.form.validate()) {
        return
      }
      authApi.userLogin({username: this.username, password: this.password})
        .then(res => {
          if (!res.data) {
            alert('登录失败')
						this.$router.back()
          }
          console.log(res.data)
          this.login(res.data)
          this.dialog = false
					this.$notify({
						group: 'user',
						title: '登录成功'
					});
          // alert('登录成功')
					this.$router.back()
        })
        .catch(res => {
					this.$notify({
						group: 'user',
						title: '登录失败'
					});
          console.log(res)
        })
    },
    cancel () {
			this.$router.back()
    }
  },
  mounted () {
    // alert('hello')
  }
}
</script>

<style scoped>

</style>
