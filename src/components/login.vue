<template>
	<div>
		<toolbar/>
		<v-container grid-list-md text-xs-center>
			<!-- <errorMessage v-if="errorMessage" :message="message"/> -->
			<v-layout justify-center>
				<v-flex xs12 sm6 md6>
					<v-card-title class="gray">
						<span class="headline black--text">Bienvenido!</span>
					</v-card-title>
					<v-form ref="form" lazy-validation>
						<v-flex xs12 sm10 md10>
							<v-text-field label="Correo Electrónico" v-model="email" :rules="emailRules" required ></v-text-field>
						</v-flex>
						<v-flex xs12 sm10 md10>
							<v-text-field :type="show ? 'text' : 'password'" v-model="password" :append-icon="show ? 'visibility_off' : 'visibility'" :rules="passwordRules" label="Contraseña" @click:append="show = !show" required ></v-text-field>
						</v-flex>
						<v-btn :disabled="!valid" @click="login">Entrar</v-btn>
						<v-btn @click="clear">Limpiar Datos</v-btn>
					</v-form>
					<googleButton/>
					<br>
					<br>
					<span>No tienes una cuenta?, </span><router-link to="/registro">Registrate Ahora</router-link>
				</v-flex>
			</v-layout>
		</v-container>
	</div>
</template>

<script>
import toolbar from '@/components/toolBar'
import {db} from '../main'
import EventBus from '@/components/EventBus'
import googleButton from '@/components/signInGoogle'

export default {
  name: 'login',
  data () {
    return {
      show: false,
      email: '',
      password: '',
      errorMessage: false,
      message: null,
      // url: 'https://carinag-225014.appspot.com',
      passwordRules: [v => !!v || 'Contraseña requerida'],
      emailRules: [
        v => !!v || 'Se requiere una cuenta de correo electrónico.',
        v => /.+@.+/.test(v) || 'Correo electrónico invalido.'
      ]
    }
  },
  methods: {
    clear () {
      this.$refs.form.reset()
    },
    login () {
      EventBus.$emit('loading', {loading: true})
      this.$store.commit('app/loginData', { email: this.email, password: this.password })
      db.collection('users').where('email', '==', this.email).where('password', '==', this.password).get().then((doc) => {
        if (doc.docs[0]) {
          let user = doc.docs[0]
          try {
            this.$store.dispatch('app/login', {data: user.data(), id: user.id})
          } finally {
            this.clear()
            EventBus.$emit('loading', {loading: false})
            this.$router.push('/')
          }
        } else {
          EventBus.$emit('loading', {loading: false})
          EventBus.$emit('errorMessage', { text: 'Usuario o contraseña invalido.', title: 'Error de Usuario', boolean: true })
        }
      }).catch(error => EventBus.$emit('errorMessage', { text: `Error al intentar verificar el usuario: ${error}`, title: 'Error en la base de datos', boolean: true }))
    }
  },
  computed: {
    valid () {
      if (this.email !== '' && /.+@.+/.test(this.email) === true && this.password !== '') {
        return true
      } else {
        return false
      }
    }
  },
  components: { toolbar, EventBus, googleButton }
}
</script>

<style lang="css">
</style>
