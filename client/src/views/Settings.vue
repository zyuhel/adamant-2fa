<template>
  <div>
    <h1>Settings</h1>
    <fieldset>
      <legend>ADAMANT</legend>
      <label>
        ADAMANT address to receive 2FA codes
        <input @input="checkAdamantAddress" autocomplete="on" maxlength="21"
         minlength="21" pattern="U\d+" v-model="account.adamantAddress"/>
        <div class="note">{{note.adamantAddress}}</div>
      </label>
      <button @click="postAdamantAddress">Get 2FA code</button>
      <br/>
      <label>
        Enter the 2FA code you received
        <input v-model="hotp"/>
      </label>
      <button @click="verifyHotp">Verify 2FA code</button>
      <div class="note">{{note.hotp}}</div>
    </fieldset>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters([
      'sessionTimeLeft'
    ])
  },
  data () {
    return {
      ...this.$store.state, // Change by commiting a mutation or dispatching an action only
      hotp: null,
      note: {
        adamantAddress: null,
        hotp: null
      }
    }
  },
  methods: {
    checkAdamantAddress (e) {
      const state = e.target.validity
      if (!state.valid) {
        // Firefox and IE have no own properties on the ValidityState object
        this.note.adamantAddress = Object.keys(Object.getPrototypeOf(state)).find(
          key => state[key]
        )
      } else this.note.adamantAddress = ''
    },
    postAdamantAddress () {
      this.axios.post(this.apiUrl + 'adamantAddress?access_token=' + this.session.id,
        this.account
      )
        .then(res => {
          if (res.status === 200) {
            this.$store.commit('UPDATE_ACCOUNT', {
              adamantAddress: res.data.adamantAddress
            })
            this.hotp = String(res.data.hotp) // @todo Get HOTP from ADAMANT messenger
            console.info(res)
          } else console.warn(res)
        })
        .catch(err => console.error(err))
    },
    verifyHotp () {
      this.axios.get(this.apiUrl + 'verifyHotp', {
        params: {
          access_token: this.session.id,
          id: this.account.id,
          hotp: this.hotp
        }
      })
        .then(res => {
          if (res.data === true) {
            this.note.hotp = '2FA succesfully enabled' // @todo Get notes from response
          } else {
            this.note.hotp = '2FA code is not valid'
          }
          console.info(res)
        })
        .catch(err => console.error(err))
    }
  }
}
</script>

<style scoped>
.note {color: red}
</style>