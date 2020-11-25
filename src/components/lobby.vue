<template>
  <div align-center justify-center style="border-radius: 20px; border: 1px solid #f5f5f5;box-shadow: 5px 2px 5px 5px #ebebeb;" class="p-1 mr-4">
      <v-row no-gutters align="center" >
        <v-col cols="12" md="8" xs="12" sm="12">
          <v-row no-gutters>
            <v-col cols="12" md="12">
              <video width="100%" height="100%" style="border-radius: 20px;">
                Your browser does not support the video tag.
              </video>
            </v-col>
          </v-row>
          <v-row no-gutters>
            <v-col cols="12" md="12" class="text-center">
              <v-btn
              depressed
              color="error"
              fab
              @click="closeAudio"
              dark
              class="mr-4">
              <v-icon v-if="clickvoice">mdi-microphone-off</v-icon>
              <v-icon v-if="!clickvoice">mdi-microphone</v-icon>
            </v-btn>
            <v-btn
            depressed
              color="error"
              @click="closeVideo"
              fab
              dark>
              <v-icon v-if="clickvideo">mdi-video-off</v-icon>
              <v-icon v-if="!clickvideo">mdi-video</v-icon>
            </v-btn>
            </v-col>
          </v-row>
        </v-col>
        <v-col cols="12" md="4" xs="12" sm="12" class="text-center">
          <v-row no-gutters>
            <v-col cols="12" md="12" xs="12" sm="12">
              <div style="font-size: 28px;">การประชุมพร้อมแล้ว</div>
            </v-col>
          </v-row>
          <v-row no-gutters>
            <v-col>
              <div class="mt-1">
                {{ urlJoin }}
                <v-btn icon @click="copyLink">
                  <v-icon left style="font-size: 18px;">
                    mdi-image-filter-none
                  </v-icon>
                </v-btn>
                <input type="hidden" v-model="txtLink" id="txtLink" ref="txtLink" />
              </div>
            </v-col>
          </v-row>
            <v-row no-gutters
              class="mt-2"
              align="center"
            >
              <v-col cols="12" md="12" xs="12" sm="12" class="pb-6 pt-6" align="center" v-show="!$vuetify.breakpoint.xs">
                <v-btn
                  @click="startConference"
                  :loading="loading"
                  color="#006000"
                  class="pt-7 pb-7"
                  style="color: #FFFFFF; font-size: 14pt; border-radius:10px;"
                >
                <v-icon class="pr-1">mdi-archive-arrow-up-outline</v-icon>
                  เข้าร่วมประชุม
                </v-btn>
              </v-col>
              <!-- <v-col cols="12" md="6" xs="12" sm="12" align="left" class="pl-2" v-show="!$vuetify.breakpoint.xs">
                <v-btn
                  class="pt-7 pb-7 pl-7 pr-7"
                  @click="closeVideo()"
                  style="color: black; padding: 24px; font-size: 14pt; border-radius:10px; "
                >
                  <v-icon class="pr-1">mdi-arrow-left-box</v-icon>
                    กลับหน้าหลัก
                </v-btn>
              </v-col> -->
            </v-row>
              <!-- xs -->
              <v-row
              no-gutters
              class="mt-2"
              align="center"
              justify="space-around">
              <v-col cols="12" md="6" xs="12" sm="12" class="pb-6 pt-6" align="center" v-show="$vuetify.breakpoint.xs">
                <v-btn
                  @click="startConference"
                  :loading="loading"
                  color="#006000"
                  class="pt-7 pb-7"
                  style="color: #FFFFFF; font-size: 14pt; border-radius:10px;"
                >
                <v-icon class="pr-1">mdi-archive-arrow-up-outline</v-icon>
                  เข้าร่วมประชุม
                </v-btn>
              </v-col>
              <!-- <v-col cols="12" md="6" xs="12" sm="12" align="center" class="pl-2" v-show="$vuetify.breakpoint.xs">
                <v-btn
                  class="pt-7 pb-7 pl-7 pr-7"
                  @click="closeVideo()"
                  style="color: black; padding: 24px; font-size: 14pt; border-radius:10px; "
                >
                  <v-icon class="pr-1">mdi-arrow-left-box</v-icon>
                    กลับหน้าหลัก
                </v-btn>
              </v-col> -->
              <!-- xs -->
            </v-row>
        </v-col>
      </v-row>
      <v-dialog
    v-model="dialog"
    width="400"
  >
    <v-card>
      <v-card-title class="grey lighten-2">
        แจ้งเตือน
      </v-card-title>

      <v-spacer></v-spacer>

      <v-card-text>
        เนื่องจาก เจ้าของห้อง ไม่อนุญาติให้เข้าห้อง
      </v-card-text>

      <v-divider></v-divider>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          color="primary"
          text
          @click="callback"
        >
          ตกลง
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import io from 'socket.io-client'

export default {
  mounted () {
    this.getMedia()
    this.getApprove()
    this.sockets()
  },
  data () {
    return {
      mediaStream: null,
      clickvideo: false,
      clickvoice: false,
      // url: this.$cookies.get('meetUrl'),
      datapage: '',
      jwt: require('jsonwebtoken'),
      urlJoin: '',
      uuid: '',
      password: '',
      prevRoute: null,
      txtLink: '',
      loading: false,
      dialog: false,
      approve: false,
      meetingId: '',
      userIdfromURL: '',
      socket: ''
    }
  },
  beforeRouteEnter (to, from, next) {
    // console.log('otp', next)
    next(vm => {
      vm.prevRoute = from
    })
  },
  computed: {
    ...mapState('dialog', ['displayName']),
    localDisplayName: {
      get () {
        return this.displayName
      },
      set (value) {
        this.setDisplayName(value)
      }
    }
  },
  methods: {
    ckickvideo () {
    },
    sockets () {
      const id = this.datapage.user._id
      this.socket.client.on(id, (response) => {
        this.loading = false
        if (response.approve === true) {
          window.open(this.$cookies.get('meetUrl'), '_self')
        } else if (response.approve === false) {
          this.dialog = true
        }
      })
    },
    getApprove () {
      var data = {
        meetingid: this.meetingid
      }
      // http://localhost:9213/api/avatar/pictureprofile
      this.axios
        .post(process.env.VUE_APP_API + '/api/rooms/getApprove', data)
        .then(res => {
          if (res.data.status === 'success') {
            this.approve = res.data.approve
          }
        })
    },
    getApproveByUser () {
      var data = {
        meeting_id: this.meetingid,
        user_id: this.datapage.user._id
      }
      // http://localhost:9213/api/avatar/pictureprofile
      this.axios
        .post(process.env.VUE_APP_API + '/api/rooms/getApproveByUser', data)
        .then(res => {
          if (res.data.status === 'success') {
            this.approveUser = res.data.approve
          }
        })
    },
    getMedia () {
      this.socket = io.connect('http://localhost:9000')
      var url = this.$cookies.get('meetUrl')
      this.meetingId = url.split('/')[3].split('?')[0]
      this.userIdfromURL = url.split('?')[6]
      // console.log('url = ' + url)
      // console.log('Id = ' + this.datapage.user._id)
      // console.log('hostId = ' + this.hostId)
      // Prefer camera resolution nearest to 1280x720.
      var constraints = { audio: true, video: { width: 1280, height: 650 } }
      const vm = this
      navigator.mediaDevices
        .getUserMedia(constraints)
        .then(function (mediaStream) {
          var video = document.querySelector('video')
          vm.mediaStream = mediaStream
          video.srcObject = mediaStream
          //   console.log(mediaStream)
          //   console.log(video.srcObject)
          video.onloadedmetadata = function (e) {
            video.play()
          }
        })
        .catch(function (err) {
          console.log(err.name + ': ' + err.message)
        }) // always check for errors at the end.
    },
    goPage (link) {
      this.$router.push(link).catch(() => {})
    },
    closeVideo () {
      if (this.clickvideo === true) {
        // console.log(this.mediaStream)
        if (this.mediaStream) {
          this.mediaStream.getTracks().forEach(function (track) {
          // track.stop()
          // console.log(track)
            if (track.kind === 'video') {
              track.enabled = true
              // console.log('start: ', track)
            }
          })
        }
      } else {
        if (this.mediaStream) {
          // console.log(this.mediaStream)
          this.mediaStream.getTracks().forEach(function (track) {
          // track.stop()
          // console.log(track)
            if (track.kind === 'video') {
              track.enabled = false
              // console.log('stop: ', track)
            }
          })
        }
      }
      this.clickvideo = !this.clickvideo
    },
    closeAudio () {
      if (this.clickvoice === true) {
        if (this.mediaStream) {
          this.mediaStream.getTracks().forEach(function (track) {
          // track.stop()
          // console.log(track)
            if (track.kind === 'audio') {
              track.enabled = true
            }
          })
        }
      } else {
        if (this.mediaStream) {
          this.mediaStream.getTracks().forEach(function (track) {
          // track.stop()
          // console.log(track)
            if (track.kind === 'audio') {
              track.enabled = false
            }
          })
        }
      }
      this.clickvoice = !this.clickvoice
    },
    startConference () {
      if (this.approve && !this.approveUser && this.userIdfromURL !== this.datapage.user._id) {
        this.socket.client.emit('lobbyApprove', {
          meetingId: this.meetingId,
          name: this.datapage.user.username,
          id: this.datapage.user._id
        })
        this.loading = true
      } else {
        window.open(this.$cookies.get('meetUrl'), '_self')
      }
      // if (this.$cookies.get('meetUrl') !== null) {
      //   window.open(this.$cookies.get('meetUrl'), '_self')
      // } else {
      //   this.joinroom()
      // }
      // window.open(this.$cookies.get('meetUrl'), '_self')
    },
    callback () {
      this.$router.push('/')
    },
    ckuuId () {
      if (this.$route.query.uuid !== undefined) {
        this.urlJoin = process.env.VUE_APP_DOMAIN + '/join/?uuid=' + this.$route.query.uuid
        this.txtLink = this.urlJoin
      } else {
        this.urlJoin = process.env.VUE_APP_DOMAIN + '/join/?uuid=' + this.datapage.room.uid
        this.txtLink = this.urlJoin
      }
    },
    decodejwt () {
      if (this.$cookies.get('user-token') !== null) {
        this.d = this.$cookies.get('user-token')
        const decryptedText = this.CryptoJS.AES.decrypt(this.d, 'One Conference').toString(this.CryptoJS.enc.Utf8)
        var data = this.jwt.decode(decryptedText)
        this.datapage = data
        // console.log('datapage', this.datapage)
      }
    },
    async joinNoLogin () {
      this.uuid = this.$route.query.uuid
      try {
        var { data } = await this.axios.post(process.env.VUE_APP_API + '/api/rooms/unauth/joinroom', {
          roomuid: this.uuid,
          name: this.localDisplayName,
          key: this.password
        })
        // console.log(data)
        if (data.status === 'success') {
          window.open(data.url, '_self')
          // this.$cookies.set('meetUrl', data.url)
          // var url = '/lobby?uuid=' + this.$route.query.uuid
          // this.$router.push(url)
        } else {
          this.$swal('WARNING !', data.message, 'warning')
        }
      } catch (error) {
        this.$swal('ERROR !', error.message, 'error')
      }
    },
    joinroom () {
      var url = this.$cookies.get('meetUrl')
      window.open(url, '_self')
    },
    copyLink () {
      var txtLink = document.getElementById('txtLink')
      txtLink.type = 'text'
      txtLink.select()
      txtLink.setSelectionRange(0, 99999) /* For mobile devices */
      document.execCommand('copy')
      txtLink.type = 'hidden'
      // console.log(cid)
    }
  },
  watch: {
    overlay (val) {
      val && setTimeout(() => {
        this.overlay = false
      }, 1000)
    }
  },
  beforeDestroy () {
    if (this.mediaStream) {
      this.mediaStream.getTracks().forEach(function (track) {
        track.stop()
      })
    }
  },
  created () {
    this.decodejwt()
    this.ckuuId()
  }
}
</script>
