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
      this.message = 'データ取得準備中...'
      const date = Date.now()
      const difficultyLevel = ['Basic', 'Advanced', 'Expert', 'Master', 'ReMaster']
      let scoreData = []
      for (let i = 0; i < difficultyLevel.length; i++) {
        const docs = await db
          .collection('users')
          .doc('l6aWFHMbrNhDnvLsHsrH')
          .collection('scores')
          .doc(difficultyLevel[i])
          .get()
        let gotOldScore
        if (docs && docs.exists) {
          gotOldScore = docs.data()
        }
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

            const imgList = classList[j].getElementsByClassName('h_30 f_r')
            let rank = null
            let comboRank = null
            let sync = null
            for (let k = 0; k < imgList.length; k++) {
              if (imgList[k].src.indexOf('_fc.png') >= 0) {
                comboRank = 'FC'
              } else if (imgList[k].src.indexOf('_fcp.png') >= 0) {
                comboRank = 'FC+'
              } else if (imgList[k].src.indexOf('_ap.png') >= 0) {
                comboRank = 'AP'
              } else if (imgList[k].src.indexOf('_app.png') >= 0) {
                comboRank = 'AP+'
              } else if (imgList[k].src.indexOf('_d.png') >= 0) {
                rank = 'D'
              } else if (imgList[k].src.indexOf('_c.png') >= 0) {
                rank = 'C'
              } else if (imgList[k].src.indexOf('_b.png') >= 0) {
                rank = 'B'
              } else if (imgList[k].src.indexOf('_bb.png') >= 0) {
                rank = 'BB'
              } else if (imgList[k].src.indexOf('_bbb.png') >= 0) {
                rank = 'BBB'
              } else if (imgList[k].src.indexOf('_a.png') >= 0) {
                rank = 'A'
              } else if (imgList[k].src.indexOf('_aa.png') >= 0) {
                rank = 'AA'
              } else if (imgList[k].src.indexOf('_aaa.png') >= 0) {
                rank = 'AAA'
              } else if (imgList[k].src.indexOf('_s.png') >= 0) {
                rank = 'S'
              } else if (imgList[k].src.indexOf('_ss.png') >= 0) {
                rank = 'SS'
              } else if (imgList[k].src.indexOf('_sss.png') >= 0) {
                rank = 'SSS'
              } else if (imgList[k].src.indexOf('_sp.png') >= 0) {
                rank = 'S+'
              } else if (imgList[k].src.indexOf('_ssp.png') >= 0) {
                rank = 'SS+'
              } else if (imgList[k].src.indexOf('_sssp.png') >= 0) {
                rank = 'SSS+'
              } else if (imgList[k].src.indexOf('_fs.png') >= 0) {
                sync = 'FS'
              } else if (imgList[k].src.indexOf('_fsd.png') >= 0) {
                sync = 'FDX'
              }
            }

            const type = classList[j].lastElementChild.src.indexOf('standard.png') >= 0 ? 'standard' : 'deluxe'
            let oldAchievement = []
            let oldDxScore = []
            if (gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`]) {
              oldAchievement = gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`].achievement || []
              oldDxScore = gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`].dxScore || []
            }
            if ((oldAchievement.length >= 1 && oldAchievement[oldAchievement.length - 1].score !== Number(tmp[2].replace('%', ''))) || (oldAchievement.length === 0 && tmp[2])) {
              oldAchievement.push({ score: Number(tmp[2].replace('%', '')), date: date })
            }
            if ((oldDxScore.length >= 1 && oldDxScore[oldDxScore.length - 1].score !== Number(tmp[3].replace(',', ''))) || (oldDxScore.length === 0 && tmp[3])) {
              oldDxScore.push({ score: Number(tmp[3].replace(',', '')), date: date })
            }

            const achievement = tmp[2] ? oldAchievement : null
            const dxScore = tmp[3] ? oldDxScore : null
            scoreData[difficultyLevel[i]][`${tmp[1]}_${difficultyLevel[i]}_${type}`] = {
              title: tmp[1],
              achievement: achievement,
              dxScore: dxScore,
              type: type,
              genre: genre,
              difficultyLevel: difficultyLevel[i],
              level: tmp[0].indexOf('+') >= 0 ? Number(tmp[0].replace('+', '')) + 0.5 : Number(tmp[0].replace('+', '')),
              rank: rank,
              comboRank: comboRank,
              sync: sync,
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
