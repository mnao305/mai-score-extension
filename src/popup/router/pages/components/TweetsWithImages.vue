<template>
  <a href="" @click.prevent="updateScoreTweet">画像付きスコア更新ツイート</a>
</template>

<script>
import firebase from '../../../plugins/firebase'
import axios from 'axios'

export default {
  data() {
    return {}
  },
  methods: {
    async updateScoreTweet() {
      this.$emit('tweetStatusUpdate', 'ツイート中です...')
      try {
        const token = await firebase.auth().currentUser.getIdToken()

        await axios.get('https://us-central1-mai-score.cloudfunctions.net/tweet', {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        })
        this.$emit('tweetStatusUpdate', 'ツイート成功しました！')
      } catch (error) {
        console.error(error)
        this.$emit('tweetStatusUpdate', 'ツイート失敗しました...')
      }
    },
  },
}
</script>
