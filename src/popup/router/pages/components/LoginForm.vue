<template>
  <div id="loginForm">
    <button @click="twitterLogin">Twitterログイン</button><br />
    <label>メールアドレス<br /><input type="text" name="mail" id="mail" v-model="mail"/></label><br />
    <label>パスワード<br /><input type="password" name="passwd" id="passwd" v-model="passwd"/></label><br />
    <button @click="mailLogin">メールログイン</button>
    <p class="errorMessage">{{ errorMessage }}</p>
  </div>
</template>

<script>
import auth from '../../../plugins/auth'
import { db } from '../../../plugins/firestore'
export default {
  data() {
    return {
      errorMessage: '',
      mail: '',
      passwd: '',
    }
  },
  methods: {
    async twitterLogin() {
      try {
        const data = await auth.twitterLogin()
        const dbData = await db
          .collection('users')
          .doc(data.user.uid)
          .get()

        if (!dbData.exists) {
          // DBにデータがなければ新規ユーザとしエラーを吐く
          this.errorMessage = '先にWebページにて新規登録してください。'
          await auth.logout()
          return
        }

        await db
          .collection('users')
          .doc(dbData.id)
          .collection('secure')
          .doc(dbData.id)
          .set(
            {
              accessToken: data.credential.accessToken,
              secret: data.credential.secret,
            },
            { merge: true }
          )

        location.reload()
      } catch (error) {
        this.errorMessage = 'ログインに失敗しました。'
      }
    },
    async mailLogin() {
      try {
        await auth.mailLogin(this.mail, this.passwd)
        location.reload()
      } catch (error) {
        this.errorMessage = 'ログインに失敗しました。'
      }
    },
  },
}
</script>

<style lang="scss" scoped>
#loginForm {
  .errorMessage {
    color: red;
  }
  button,
  input {
    margin-bottom: 10px;
  }
  input {
    width: 260px;
  }
}
</style>
