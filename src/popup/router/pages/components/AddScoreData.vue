<template>
  <div id="addScoreData">
    <a id="logout" href @click="logout">ログアウト</a>
    <p>舞スコア データ取得ツール</p>
    <button :disabled="isDisable" class="addDataBtn" :class="{ disableBtn: isDisable }" @click="getData">データ取得</button>
    <p v-if="message" :class="{ error: error }">{{ message }}</p>
    <p v-if="publicData"><a :href="tweetURL" target="_blank">スコア更新ツイートする</a></p>
  </div>
</template>

<script>
import Axios from 'axios'
import { db } from '../../../plugins/firestore'
import auth from '../../../plugins/auth'
import firebase from '../../../plugins/firebase'

export default {
  data() {
    return {
      message: '',
      error: false,
      publicData: false,
      tweetURL: '',
      isDisable: false,
    }
  },
  props: {
    uid: String,
  },
  methods: {
    async getData() {
      this.error = false
      this.isDisable = true
      this.message = 'データ取得準備中...'
      const date = Date.now()
      const difficultyLevel = ['Basic', 'Advanced', 'Expert', 'Master', 'ReMaster']
      let scoreData = []
      let updateScoreData = []
      for (let i = 0; i < difficultyLevel.length; i++) {
        const docs = await db
          .collection('users')
          .doc(this.uid)
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
          if (data.match(/ログインしてください/)) {
            this.message = 'maimaiでらっくすNETにログインしていません。ログインしてから再度お試しください。'
            this.error = true
            this.isDisable = false
            return
          }
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
              } else if (imgList[k].src.indexOf('_fsp.png') >= 0) {
                sync = 'FS+'
              } else if (imgList[k].src.indexOf('_fsdp.png') >= 0) {
                sync = 'FDX+'
              }
            }

            let musicUpdateDate

            const type = classList[j].lastElementChild.src.indexOf('standard.png') >= 0 ? 'standard' : 'deluxe'
            let oldAchievement = []
            let oldDxScore = []
            if (gotOldScore && gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`]) {
              oldAchievement = gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`].achievements || []
              oldDxScore = gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`].dxScores || []
              musicUpdateDate = gotOldScore[`${tmp[1]}_${difficultyLevel[i]}_${type}`].date || date
            } else {
              musicUpdateDate = date
            }
            let updateFlg = false
            if (
              (oldAchievement.length >= 1 && oldAchievement[oldAchievement.length - 1].achievement !== Number(tmp[2].replace('%', ''))) ||
              (oldAchievement.length === 0 && tmp[2])
            ) {
              oldAchievement.push({ achievement: Number(tmp[2].replace('%', '')), date: date })
              musicUpdateDate = date
              updateFlg = true
            }
            if ((oldDxScore.length >= 1 && oldDxScore[oldDxScore.length - 1].dxScore !== Number(tmp[3].replace(',', ''))) || (oldDxScore.length === 0 && tmp[3])) {
              oldDxScore.push({ dxScore: Number(tmp[3].replace(',', '')), date: date })
              musicUpdateDate = date
              updateFlg = true
            }

            const achievements = tmp[2] ? oldAchievement : null
            const dxScores = tmp[3] ? oldDxScore : null
            scoreData[difficultyLevel[i]][`${tmp[1]}_${difficultyLevel[i]}_${type}`] = {
              title: tmp[1],
              achievements: achievements,
              dxScores: dxScores,
              type: type,
              genre: genre,
              difficultyLevel: difficultyLevel[i],
              level: tmp[0].indexOf('+') >= 0 ? Number(tmp[0].replace('+', '')) + 0.5 : Number(tmp[0].replace('+', '')),
              rank: rank,
              comboRank: comboRank,
              sync: sync,
              date: musicUpdateDate,
              musicID: classList[j].getElementsByTagName('input')[0].value,
            }
            if (updateFlg) {
              updateScoreData.push(scoreData[difficultyLevel[i]][`${tmp[1]}_${difficultyLevel[i]}_${type}`])
            }
          }
        } catch (error) {
          console.error(error.data)
          if (error.response && error.response.data && error.response.data.match(/メンテナンス中/)) {
            this.message = 'maimaiでらっくすNETはメンテナンス中です。メンテナンス終了後に再度お試しください。'
            this.error = true
            this.isDisable = false
            return
          }
          continue
        }
      }
      console.log(scoreData)
      await this.getFetchUserData(date)
      this.message = 'データ保存中...'

      // テスト用に一旦保存停止
      // for (let i = 0; i < difficultyLevel.length; i++) {
      //   db.collection('users')
      //     .doc(this.uid)
      //     .collection('scores')
      //     .doc(difficultyLevel[i])
      //     .set(scoreData[difficultyLevel[i]], { merge: true })
      //     .catch(e => {
      //       console.error(e)
      //     })
      // }
      await db
        .collection('users')
        .doc(this.uid)
        .collection('secure')
        .doc(this.uid)
        .update({
          _updateAt: date,
        })
      this.getTweetURL()
      this.createScoreImg(updateScoreData)
      this.message = 'データ保存完了！'
    },
    async getFetchUserData(date) {
      this.message = 'ユーザデータを読み込み中...'
      try {
        const { data } = await Axios.get('https://maimaidx.jp/maimai-mobile/playerData/')
        const element = document.createElement('div')
        element.innerHTML = data
        if (data.match(/ログインしてください/)) {
          this.message = 'maimaiでらっくすNETにログインしていません。ログインしてから再度お試しください。'
          this.error = true
          this.isDisable = false
        }
        const gotRating = element.getElementsByClassName('rating_block f_11')[0].innerText
        const gotMaxRating = Number(element.getElementsByClassName('p_r_5 f_11')[0].innerText.split('：')[1])
        const gotPlayCount = Number(
          element
            .getElementsByClassName('m_5 m_t_10 t_r f_12')[0]
            .innerText.split('：')[1]
            .split('回')[0]
        )
        const gotUserName = element.getElementsByClassName('name_block f_l f_14')[0].innerText.replace(/[Ａ-Ｚａ-ｚ０-９]/g, s => {
          return String.fromCharCode(s.charCodeAt(0) - 65248)
        })
        const docs = await db
          .collection('users')
          .doc(this.uid)
          .collection('userData')
          .doc(this.uid)
          .get()
        let gotOldUserData
        if (docs && docs.exists) {
          gotOldUserData = docs.data()
        }
        let ratings = []
        if (gotOldUserData) {
          ratings = gotOldUserData.ratings || []
        }
        if ((ratings.length >= 1 && ratings[ratings.length - 1].rating !== Number(gotRating)) || ratings.length === 0) {
          ratings.push({ rating: Number(gotRating), date: date })
        }
        await db
          .collection('users')
          .doc(this.uid)
          .collection('userData')
          .doc(this.uid)
          .set(
            {
              ratings: ratings,
              maxRating: gotMaxRating,
              playCount: gotPlayCount,
              userName: gotUserName,
            },
            { merge: true }
          )
      } catch (error) {
        console.error(error)

        if (error.response && error.response.data && error.response.data.match(/メンテナンス中/)) {
          this.message = 'maimaiでらっくすNETはメンテナンス中です。メンテナンス終了後に再度お試しください。'
          this.error = true
          this.isDisable = false
        }
      }
    },
    async logout() {
      await auth.logout()
      location.reload()
    },
    async getTweetURL() {
      const doc = await db
        .collection('users')
        .doc(this.uid)
        .get()
      const userData = doc.data()
      this.publicData = userData.public
      this.tweetURL = `https://twitter.com/intent/tweet?text=スコア更新しました！&hashtags=舞スコア&url=https://mai-score.com/user/${userData.displayName}`
    },
    async createScoreImg(updateScoreData) {
      console.log(updateScoreData)
      updateScoreData.reverse()

      if (updateScoreData.length >= 10) {
        updateScoreData = updateScoreData.slice(0, 10)
      }

      let canvas = document.createElement('canvas')
      canvas.width = 600
      canvas.height = 80 * updateScoreData.length
      canvas.style.width = 600
      canvas.style.height = 80 * updateScoreData.length
      let ctx = canvas.getContext('2d')
      ctx.fillStyle = 'white'
      ctx.fillRect(0, 0, 600, 600)
      ctx.fillStyle = 'black'
      updateScoreData.forEach((v, i) => {
        console.log(v)
        this.saveMusicIcon(v.musicID)
        ctx.font = `24px 'Meiryo'`
        const level = Math.round(v.level) === v.level ? v.level : `${v.level - 0.5}+`
        ctx.fillText(`[${v.difficultyLevel}|${level}]${v.title}`, 10, 30 * (i * 2 + 1) + 15 * i)
        let text
        let dxText
        if (v.achievements.length >= 2) {
          text = `${v.achievements.slice(-2)[0].achievement}%→${v.achievements.slice(-1)[0].achievement}%  +${(
            v.achievements.slice(-1)[0].achievement - v.achievements.slice(-2)[0].achievement
          ).toFixed(4)}%`
          dxText = `${v.dxScores.slice(-2)[0].dxScore}→${v.dxScores.slice(-1)[0].dxScore}  +${v.dxScores.slice(-1)[0].dxScore - v.dxScores.slice(-2)[0].dxScore}`
        } else {
          text = `0.0000%→${v.achievements.slice(-1)[0].achievement}%  +${v.achievements[0].achievement.toFixed(4)}%`
          dxText = `0→${v.dxScores.slice(-1)[0].dxScore}  +${v.dxScores[0].dxScore}`
        }
        ctx.font = `16px 'Meiryo'`
        ctx.fillText(text, 10, 30 * (i * 2 + 1) + 20 + 15 * i)
        ctx.fillText(dxText, 10, 30 * (i * 2 + 1) + 40 + 15 * i)
        ctx.fillText(`${v.rank}  ${v.sync || ''}`, 200, 30 * (i * 2 + 1) + 40 + 15 * i)
      })
      const imgUrl = canvas.toDataURL('image/jpeg')
      // Firebase Storageに画像をアップロード
      try {
        const storageRef = firebase
          .storage()
          .ref(`userData/${this.uid}/`)
          .child('updateScore.jpg')
        const data = await storageRef.putString(imgUrl, 'data_url')
        console.log(data)
      } catch (error) {
        console.error(error)
      }
    },
    async saveMusicIcon(musicID) {
      const { data } = await Axios.get(`https://maimaidx.jp/maimai-mobile/record/musicDetail/?idx=${encodeURIComponent(musicID)}`)
      const tmpEl = document.createElement('div')
      tmpEl.innerHTML = data

      const title = tmpEl.getElementsByClassName('m_5 f_15 break')[0].innerText
      const musicImgUrl = tmpEl.getElementsByClassName('w_180 m_5 f_l')[0].src
      const musicIcon = await Axios.get(musicImgUrl, { responseType: 'arraybuffer' })
      console.log(title)
      try {
        const storageRef = firebase
          .storage()
          .ref('musicIcon/')
          .child(`${title}.png`)
        const data = await storageRef.put(musicIcon.data, {
          contentType: 'image/png',
        })
        console.log(data)
      } catch (error) {
        console.error(error)
      }
    },
  },
}
</script>

<style lang="scss" scoped>
#addScoreData {
  .error {
    color: red;
  }
  #logout {
    display: block;
    text-align: right;
    margin: 0;
  }
  .addDataBtn {
    padding: 0.2em 0.5em;
    text-decoration: none;
    color: #0097a7;
    border: solid 2px #0097a7;
    border-radius: 3px;
    transition: 0.4s;
    background: white;
    cursor: pointer;
  }
  .addDataBtn:hover {
    background: #00acc1;
    color: white;
  }
  .disableBtn {
    cursor: not-allowed;
    background: #00acc1;
    color: white;
  }
}
</style>
