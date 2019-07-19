<template>
  <div id="popupIndex">
    <add-score-data v-if="isLoginFlg" :uid="uid" />
    <login-form v-else />
  </div>
</template>

<script>
import AddScoreData from './components/AddScoreData.vue'
import LoginForm from './components/LoginForm.vue'
import auth from '../../plugins/auth'
export default {
  components: {
    AddScoreData,
    LoginForm,
  },
  data() {
    return {
      isLoginFlg: false,
      uid: null,
    }
  },
  async created() {
    const data = await auth.auth()
    if (data) {
      this.uid = data.uid
      this.isLoginFlg = true
    }
  },
}
</script>

<style lang="scss" scoped>
#popupIndex {
  width: 300px;
  text-align: center;
}
</style>
