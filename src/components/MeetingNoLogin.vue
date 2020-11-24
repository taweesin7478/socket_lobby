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
            <v-col><div class="mt-1">{{ urlJoin }}</div></v-col>
          </v-row>
           <v-row>
          <v-col class="text-center">
            <p style="#525151; font-size:39px">{{ roomName }}</p>
          </v-col>
        </v-row>
          <v-row class="mt-n12">
          <v-col class="text-center">
            <p style="#525151; font-size:15px">
              Invite by : {{fullname}}
            </p>
          </v-col>
        </v-row>
        <v-row no-gutters class="text-center mt-n2">
          <v-col cols="12" md="2"></v-col>
          <v-col cols="12" md="8" sm="10" xs="12">
            <v-text-field outlined placeholder="Enter your name" v-model="displayName"></v-text-field>
          </v-col>
        </v-row>
        <v-row no-gutters class="text-center " v-if="showPassword" >
          <v-col cols="12" md="2"  ></v-col>
          <v-col cols="12" md="8" sm="10" xs="12"  >
            <v-text-field
              outlined
              placeholder="Password meeting"
              v-model="password"
              type="password"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row class="text-center mt-n8" >
          <v-col cols="12" md="4"  ></v-col>
          <v-col cols="12" md="4" sm="10" xs="12">
            <div style="float:center">
              <v-btn
                color="#006000"
                  class="pt-7 pb-7"
                  style="color: #FFFFFF; padding: 24px; font-size: 14pt; border-radius:10px;"
                @click="joinNoLogin"
              ><v-icon class="pr-1">mdi-archive-arrow-up-outline</v-icon>
              Join</v-btn>
            </div>
          </v-col>
        </v-row>
        </v-col>
      </v-row>
  </div>
</template>
<script>
import { mapActions } from 'vuex'

export default {
  mounted () {
    this.getMedia()
  },
  data () {
    return {
      password: '',
      uuid: '',
      fullname: '',
      clickvideo: false,
      clickvoice: false,
      showPassword: false,
      displayName: '',
      roomName: '',
      key: false
    }
  },
  created () {
    // console.log('created')
    this.checkroom()
  },
  methods: {
    ...mapActions('dialog', [
      'setDisplayName'
    ]),
    setValueDisplayName () {
      this.setDisplayName(this.displayName)
      // var url = '/lobby?uuid=' + this.$route.query.uuid
      // this.$router.push(url)
      // this.$router.push('/join?uuid=' + this.$route.query.uuid)
      this.joinNoLogin()
    },
    getMedia () {
    // Prefer camera resolution nearest to 1280x720.
      var constraints = { audio: true, video: { width: 1280, height: 720 } }
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
    async checkroom () {
      this.uuid = this.$route.query.uuid
      try {
        var { data } = await this.axios.get(
          process.env.VUE_APP_API + '/api/rooms/check/' + this.$route.query.uuid
        )
        // this.roominfo = data
        // console.log('data', data)
        this.fullname = data.fullname
        this.roomName = data.roomname
        if (data.key === 'true') {
          this.showPassword = true
        } else {
          this.showPassword = false
        }
      } catch (error) {
        // this.$swal('warning!', 'meeting is not start', 'warning')
        this.$swal.fire({
          title: 'Warning !',
          text: 'meeting is not start',
          icon: 'warning',
          showCancelButton: false,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'ok'
        }).then((result) => {
          if (result.value) {
            this.$router.push('/')
          }
        })
        // window.history.back()
        // console.log('error', error.message)
      }
    },
    checkName () {
      if (this.displayName !== '') {
        this.setValueDisplayName()
      } else {
        this.$swal.fire({
          title: 'ยังไม่กรอกชื่อ !',
          text: 'กรุณากรอกชื่อ',
          icon: 'warning',
          showCancelButton: false,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'ok'
        })
      }
    },
    async joinNoLogin () {
      console.log('name', this.displayName)
      this.uuid = this.$route.query.uuid
      try {
        var { data } = await this.axios.post(process.env.VUE_APP_API + '/api/rooms/unauth/joinroom', {
          roomuid: this.uuid,
          name: this.displayName,
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
    }
  }
}
</script>
