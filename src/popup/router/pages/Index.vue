<template>
  <div id="popupIndex">
    <p>舞スコア データ取得ツール</p>
    <button @click="getData">データ取得</button>
    <p v-if="message">{{ message }}</p>
  </div>
</template>

<script>
// import Browser from 'webextension-polyfill'
import Axios from 'axios'
import { db } from '../../plugins/firestore'
export default {
  data() {
    return {
      message: '',
    }
  },
  methods: {
    async getData() {
      const difficultyLevel = ['basic', 'advanced', 'expert', 'master', 'remaster']
      let scoreData = {}
      let scoreDataDX = {}
      for (let i = 0; i < difficultyLevel.length; i++) {
        this.message = `${difficultyLevel[i]}データを読み込み中...`
        try {
          const { data } = await Axios.get(`https://maimaidx.jp/maimai-mobile/record/musicGenre/search/?genre=99&diff=${i}`)
          const tmpEl = document.createElement('div')
          tmpEl.innerHTML = data
          const classList = tmpEl.getElementsByClassName('w_450 m_15 p_r f_0')

          let scoreList = {}
          let scoreListDX = {}
          for (let i = 0; i < classList.length; i++) {
            const tmp = classList[i].innerText
              .trim()
              .split('\n')
              .map(v => v.trim())

            if (classList[i].lastElementChild.src.indexOf('standard.png') >= 0) {
              scoreList[tmp[1]] = {
                title: tmp[1],
                achievement: tmp[2] ? tmp[2] : null,
                dxScore: tmp[3] ? tmp[3] : null,
                type: 'standard',
              }
            } else {
              scoreListDX[tmp[1]] = {
                title: tmp[1],
                achievement: tmp[2] ? tmp[2] : null,
                dxScore: tmp[3] ? tmp[3] : null,
                type: 'deluxe',
              }
            }
          }
          scoreData[difficultyLevel[i]] = scoreList
          scoreDataDX[difficultyLevel[i]] = scoreListDX
        } catch (error) {
          console.error(error)
          continue
        }
      }
      console.log(scoreData)
      console.log(scoreDataDX)
      this.message = 'データ保存中...'
      for (let i = 0; i < difficultyLevel.length; i++) {
        db.collection('users')
          .doc('l6aWFHMbrNhDnvLsHsrH')
          .collection('scores')
          .doc(Object.keys(scoreData)[i])
          .collection('standard')
          .doc('list')
          .set(scoreData[Object.keys(scoreData)[i]], { merge: true })
          .catch(e => {
            console.error(e)
          })
        db.collection('users')
          .doc('l6aWFHMbrNhDnvLsHsrH')
          .collection('scores')
          .doc(Object.keys(scoreDataDX)[i])
          .collection('deluxe')
          .doc('list')
          .set(scoreDataDX[Object.keys(scoreDataDX)[i]], { merge: true })
          .catch(e => {
            console.error(e)
          })
      }
      this.message = 'データ保存完了！'
    },
  },
}
</script>

<style lang="scss" scoped>
#popupIndex {
  width: 300px;
  text-align: center;
}
</style>
