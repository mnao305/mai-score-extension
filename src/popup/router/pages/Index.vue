<template>
  <div id="popupIndex">
    <p>舞スコア データ取得ツール</p>
    <button @click="getData">データ取得</button>
  </div>
</template>

<script>
// import Browser from 'webextension-polyfill'
import Axios from 'axios'
export default {
  data() {
    return {}
  },
  methods: {
    async getData() {
      const difficultyLevel = ['basic', 'advanced', 'expert', 'master', 'remaster']
      let scoreData = {}
      for (let i = 0; i < difficultyLevel.length; i++) {
        try {
          const { data } = await Axios.get(`https://maimaidx.jp/maimai-mobile/record/musicGenre/search/?genre=99&diff=${i}`)
          const tmpEl = document.createElement('div')
          tmpEl.innerHTML = data
          const classList = tmpEl.getElementsByClassName('w_450 m_15 p_r f_0')

          let scoreList = {}
          for (let i = 0; i < classList.length; i++) {
            const tmp = classList[i].innerText
              .trim()
              .split('\n')
              .map(v => v.trim())
            scoreList[tmp[1]] = {
              level: tmp[0],
              title: tmp[1],
              achievement: tmp[2] ? tmp[2] : null,
              dxScore: tmp[3] ? tmp[3] : null,
              type: classList[i].lastElementChild.src.indexOf('standard.png') >= 0 ? 'standard' : 'deluxe',
            }
          }
          scoreData[difficultyLevel[i]] = scoreList
        } catch (error) {
          console.error(error)
          continue
        }
      }
      console.log(scoreData)
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
