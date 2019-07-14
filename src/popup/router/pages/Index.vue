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
      const difficultyLevel = ['Basic', 'Advanced', 'Expert', 'Master', 'ReMaster']
      let scoreData = []
      for (let i = 0; i < difficultyLevel.length; i++) {
        scoreData[difficultyLevel[i]] = {}
        this.message = `${difficultyLevel[i]}データを読み込み中...`
        try {
          const { data } = await Axios.get(`https://maimaidx.jp/maimai-mobile/record/musicGenre/search/?genre=99&diff=${i}`)
          const tmpEl = document.createElement('div')
          tmpEl.innerHTML = data
          const classList = tmpEl.getElementsByClassName('main_wrapper')[0].children

          let genre = ''
          for (let j = 0; j < classList.length; j++) {
            if (classList[j].className.indexOf('screw_block') >= 0) {
              genre = classList[j].innerText
              continue
            } else if (classList[j].className.indexOf('w_450') === -1) {
              continue
            }
            const tmp = classList[j].innerText
              .trim()
              .split('\n')
              .map(v => v.trim())

            const type = classList[j].lastElementChild.src.indexOf('standard.png') >= 0 ? 'standard' : 'deluxe'
            scoreData[difficultyLevel[i]][`${tmp[1]}_${difficultyLevel[i]}_${type}`] = {
              title: tmp[1],
              achievement: tmp[2] ? Number(tmp[2].replace('%', '')) : null,
              dxScore: tmp[3] ? Number(tmp[3].replace(',', '')) : null,
              type: type,
              genre: genre,
              difficultyLevel: difficultyLevel[i],
              level: tmp[0].indexOf('+') >= 0 ? Number(tmp[0].replace('+', '')) + 0.5 : Number(tmp[0].replace('+', '')),
            }
          }
        } catch (error) {
          console.error(error)
          continue
        }
      }
      console.log(scoreData)
      this.message = 'データ保存中...'

      for (let i = 0; i < difficultyLevel.length; i++) {
        db.collection('users')
          .doc('l6aWFHMbrNhDnvLsHsrH')
          .collection('scores')
          .doc(difficultyLevel[i])
          .set(scoreData[difficultyLevel[i]])
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
