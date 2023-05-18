<template>
  <div class="details">
    <div class="details_left">
      <p>课程名称：{{ title }}</p>
      <div class="details_left_time"><i class="rili"></i> {{ startTimeT }} 至 {{ endTimeT }}</div>
      <div class="play">
        <!--阿里云播放器-->
        <ali-player
          :encryptType="1"
          :vid="vid"
          :playauth="playAuth"
          v-if="playShow"
          ref="player"
          :useFlashPrism="false"
          :autoplay="true"
          :isLive="false"
          :showBuffer="true"
          :cover="cover"
          :showBarTime="'true'"
          :source="source"
          :seekTime="videoTimes"
          :disableSeek="FirstTime"
          :rePlay="false"
          @ended="endTime"
          @pause="endTime"
          height="660px"
          @play="startTime"
          :speedPlay="speedPlay"
          controlBarVisibility="always">
        </ali-player>
      </div>
    </div>
    <div class="details_right">
      <div class="info">
        <p>简介:</p>
        <p>{{ remark }}</p>
      </div>
      <div class="play_list">
        <p>课程列表</p>
        <div class="list_name" v-for="item in playList" :key="item.id">
          <a-tooltip>
            <template slot="title">
              {{ item.resourceName }}
            </template>
            <div class="list_title" :class="{ playTitle: item.id === playListId }" @click="playBtn(item)">
              <span :class="{ playIcon: item.id === playListId }"></span>{{ item.resourceName }}
            </div>
          </a-tooltip>
          <div class="list_Time">{{ item.resourceDuration }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import VueAliplayer from '@/components/play/AliPlayer.vue'
import { getPageResource, getUserResource, postSaveWatchHistory, getLastResourceId } from '@/api/student/Assignment'
export default {
  name: 'User',
  components: {
    'ali-player': VueAliplayer
  },
  data () {
    return {
      title: '',
      startTimeT: '',
      endTimeT: '',
      remark: '',
      playListId: '',
      playShow: true, // 参数加载完毕在给true
      playList: [],
      cover: '', // 视频封面
      vid: '', // 视频vid
      playAuth: '', // 鉴权地址
      source: '', // 视频地址
      videoTime: '', // 视频时间
      endTimeFlag: true, // 终止计时标记
      speedPlay: true, // 倍速播放
      FirstTime: true, // 是否首次播放
      PlaySum: null, // 视频总时间
      videoTimes: 0, // 视频上次看的位置
      isComplete: '0',
      seconds: '0',
      id: '',
      playListShow: false,
      pulay: false
    }
  },
  created () {
    this.id = this.$route.query.id
    getPageResource({
      current: '1',
      size: '-1',
      id: this.$route.query.id,
      userId: this.$store.state.user.user.id
    }).then((res) => {
      this.playList = res.data.records
      getLastResourceId({ userId: this.$store.state.user.user.id, id: this.$route.query.id }).then((rescon) => {
        this.playList.forEach((item) => {
          if (item.id === rescon.data) {
            this.playListId = item.id
            this.playListShow = true
          }
        })
        if (this.playListShow === false) {
          this.playListId = this.playList[0]?.id
          this.playListShow = true
        }
      })
    })
    this.title = this.$route.query.title
    this.startTimeT = this.$route.query.startTime
    this.endTimeT = this.$route.query.endTime
    this.remark = this.$route.query.remark
  },
  mounted () {
    // this.playShow = true
    let this_ = this
    document.addEventListener('fullscreenchange', function (e) {
      if (document.fullscreenElement) {
        document.querySelector('video').style.cssText = 'width: 100%; height: 90%;'
        console.log(document.querySelector('video').style.cssText)
      } else {
      }
    })
    let numplay = 1
    var timer = requestAnimationFrame(function fn () {
      const nodesObj = document.querySelector('.playTitle')
      numplay = numplay + 1
      if (numplay > 1000) {
        return cancelAnimationFrame(timer)
      }
      if (this_.playListShow) {
        if (!nodesObj) {
        requestAnimationFrame(fn)
      } else {
        nodesObj.click()
        return cancelAnimationFrame(timer)
      }
      } else {
        requestAnimationFrame(fn)
      }
    })
    window.addEventListener('beforeunload', function (event) {
      // 在窗口关闭之前执行的操作
      setTimeout(function () {
        // 确保事件能够被触发
        let data = {
          id: this.id,
          userId: this.$store.state.user.user.id,
          resourceId: this.playListId,
          second: this.videoTime ? this.videoTime : this.seconds
        }
        postSaveWatchHistory(data)
      }, 0)
    })
  },
  computed: {},
  watch: {
    playShow: {
      handler (val) {
        let this_ = this
        if (val) {
          var timer = requestAnimationFrame(function fn () {
            const nodesObj = document.querySelector('.prism-progress')
            if (nodesObj.length === 0) {
              requestAnimationFrame(fn)
            } else {
              if (this_.FirstTime) {
                nodesObj.classList.add('displayNo')
              } else {
                nodesObj.classList.remove('displayNo')
              }
              return cancelAnimationFrame(timer)
            }
          })
        } else {
        }
      },
      deep: true
    }
  },
  beforeDestroy () {
    let data = {
      id: this.id,
      userId: this.$store.state.user.user.id,
      resourceId: this.playListId,
      second: this.videoTime ? this.videoTime : this.seconds
    }
    postSaveWatchHistory(data)
  },

  methods: {
    getVideoData () {
      // 1、调用后台接口获取视频vid,playAuth(鉴权地址),cover(视频封面)的逻辑
      // 2、将对应的值分别赋值
      // this.$refs.player.player
      // console.log(this.$refs.player)
      // setInterval(this.setTime, 1000)
    },
    endTime () {
      this.endTimeFlag = true
    },
    startTime () {
      this.PlaySum = this.$refs.player.getDuration()
      this.endTimeFlag = false
      this.setTime()
    },
    setTime () {
      var interval = setInterval(() => {
        if (this.endTimeFlag) {
          console.log('暂停')
          clearInterval(interval)
          let data = {
            id: this.$route.query.id,
            userId: this.$store.state.user.user.id,
            resourceId: this.playListId,
            second: this.videoTime ? this.videoTime : this.seconds
          }
          postSaveWatchHistory(data)
        }
        this.videoTime = this.$refs.player?.getCurrentTime()
        // 进来后用户首次播放完毕给用户所有视频权限
        if (this.isComplete === '0') {
          if (Number(this.seconds) < this.videoTime) {
            let data = {
              id: this.$route.query.id,
              userId: this.$store.state.user.user.id,
              resourceId: this.playListId,
              second: this.videoTime,
              isComplete: '1'
            }
            postSaveWatchHistory(data).then((res) => {
              getPageResource({
                current: '1',
                size: '-1',
                id: this.$route.query.id,
                userId: this.$store.state.user.user.id
              }).then((res) => {
                this.playList = res.data.records
              })
            })
            clearInterval(interval)
          }
        }
      }, 1000)
    },
    playBtn (data) {
      if (this.pulay) {
        let datas = {
        id: this.id,
        userId: this.$store.state.user.user.id,
        resourceId: this.playListId,
        second: this.videoTime ? this.videoTime : this.seconds
      }
      postSaveWatchHistory(datas)
      } else {
        this.pulay = true
      }
      this.playShow = false
      this.playListId = data.id
      getUserResource({ userId: this.$store.state.user.user.id, id: this.$route.query.id, resourceId: data.id }).then(
        (res) => {
          this.videoTimes = res.data.lastStudyTime
          // 判断是否首次进来，如果是禁止拖拽快进
          if (res.data.isSpeed === 0) {
            this.FirstTime = true
            this.speedPlay = false
          } else {
            this.FirstTime = false
            this.speedPlay = true
          }
          this.isComplete = res.data.isComplete ? res.data.isComplete : '0'
          this.cover = data.resourcePhoto
          this.source = data.resourceUrl
          this.seconds = data.seconds
          this.playShow = true
        }
      )
    }
  },
  destroyed () {
    // 取消所有在全局作用域下设置的定时器
    for (var i = 1; i < 99999; i++) {
      window.clearInterval(i)
      window.clearTimeout(i)
    }
  }
}
</script>

<style lang="less" scoped>
.details {
  display: flex;
  justify-content: space-between;

  .details_left {
    p {
      font-size: 16px;
      font-family: Microsoft YaHei;
      font-weight: bold;
      color: #333333;
    }

    .details_left_time {
      font-size: 14px;
      font-family: Microsoft YaHei;
      font-weight: 400;
      color: #333333;

      .rili {
        margin-right: 17px;
        display: inline-block;
        width: 16px;
        height: 16px;
        background: url('../../../assets/images/rili.png');
        background-size: 100%;
        vertical-align: sub;
      }
    }

    .play {
      margin-top: 24px;
      width: 1044px;
      height: 656px;
    }
  }

  .details_right {
    .info {
      background-color: #fff;
      width: 332px;
      height: 234px;
      padding: 20px;
      font-size: 14px;
      font-family: Microsoft YaHei;
      font-weight: 400;
      color: #333333;
      overflow-y: auto;
      p {
        letter-spacing: 1px;
        line-height: 22px;
      }
    }

    .play_list {
      margin-top: 24px;
      background-color: #fff;
      width: 332px;
      height: 486px;
      overflow-y: auto !important;
      overflow: hidden;
      padding: 23px;

      p {
        font-size: 14px;
        font-family: Microsoft YaHei;
        font-weight: 400;
        color: #333333;
        height: 30px;
        border-bottom: 1px solid #88888838;
      }

      .list_name {
        margin-top: 23px;
        display: flex;
        justify-content: space-between;

        .list_title {
          cursor: pointer;
          font-size: 14px;
          width: 230px;
          font-family: Microsoft YaHei;
          font-weight: 400;
          color: #333333;
          //超出一行省略号
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
        }

        .list_Time {
          font-size: 14px;
          font-family: Microsoft YaHei;
          font-weight: 400;
          color: #909090;
          //超出一行省略号
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
        }
      }
    }
  }
}

.playIcon {
  margin-right: 17px;
  width: 18px;
  height: 16px;
  display: inline-block;
  background: url('../../../assets/images/play.png');
  background-size: 100%;
  vertical-align: sub;
}

.playTitle {
  color: #008fe0 !important;
}

/deep/.displayNo {
  display: none !important;
}
</style>
