<template>
  <div>
    <!-- 播放pcm格式语音 -->
    <section v-if="duration != '0'" @click="playVoice" class="qj_audio">
      <span class="voice-normal" :class="{ voicePlay: isPlay }"></span>
      <span> {{ duration }} </span>
    </section>
  </div>
</template>
<script>
export default {
  data() {
    return {
      audio:
        "https://qycdn.zhuoyoutech.com/qyresource/file/2020/2/27/MFLO584YPJNyQaa/680e0bc7-fdb8-4ae1-9f13-f3a0e574052c.pcm?13", //语音文件地址
      duration: "0", //语音的时长
      source: null, //AudioBuffer 对象 进行开始和停止语音
      sounds: null, //获取的音频数据
      isPlay: false //语音是否在播放
    };
  },
  mounted() {
    this.setDuration();
  },
  methods: {
    setDuration() {
      //问号后面的是语音时长 这里的语音时长 是服务器接口返回的
      let last = this.audio.split("?")[1];
      try {
        if (last != 0) {
          // 因为我们的语音 最长不会超过60秒 所以只做了秒的判断
          if (last < 10) {
            last = "0" + last;
          }
          this.duration = "00:" + last;
        }
      } catch (error) {
        console.log(error);
      }
    },

    playVoice() {
      var _this = this;

      if (this.isPlay) {
        //   关闭语音
        this.stopVoice();
      } else {
        _this.isPlay = true;

        //   播放语音
        if (this.sounds == null) {
          // 首次加载语音文件
          var req = new XMLHttpRequest();
          req.open("GET", this.audio, true);

          req.overrideMimeType("text\/plain; charset=x-user-defined");
          req.onreadystatechange = function(aEvt) {
            if (req.readyState == 4) {
              if (req.status == 200) {
                _this.sounds = req.responseText;
                _this.startVoice(_this);
              } else {
                window.ToastShow("failed loading audio");
              }
            }
          };
          req.send(null);
        } else {
          // 直接播放
          _this.startVoice(_this);
        }
      }
    },
    stopVoice() {
      this.source.stop();
      this.isPlay = false;
    },
    startVoice(_this) {
      // Stereo 这里的逻辑没有进行深入研究 直接拿过来用了
      var channels = 1;
      var audioCtx = new (window.AudioContext || window.webkitAudioContext)();

      // Create an empty two second stereo buffer at the
      // sample rate of the AudioContext
      var frameCount = _this.sounds.length / 2;

      var myAudioBuffer = audioCtx.createBuffer(channels, frameCount, 16000);
      for (var channel = 0; channel < channels; channel++) {
        var nowBuffering = myAudioBuffer.getChannelData(channel, 16, 16000);
        for (var i = 0; i < frameCount; i++) {
          // audio needs to be in [-1.0; 1.0]
          // for this reason I also tried to divide it by 32767
          // as my pcm sample is in 16-Bit. It plays still the
          // same creepy sound less noisy.
          var word =
            (_this.sounds.charCodeAt(i * 2) & 0xff) +
            ((_this.sounds.charCodeAt(i * 2 + 1) & 0xff) << 8);
          nowBuffering[i] = (((word + 32768) % 65536) - 32768) / 32768.0;
        }
      }
      // Get an AudioBufferSourceNode.
      // This is the AudioNode to use when we want to play an AudioBuffer
      var sources = audioCtx.createBufferSource();
      // set the buffer in the AudioBufferSourceNode
      sources.buffer = myAudioBuffer;
      // connect the AudioBufferSourceNode to the
      // destination so we can hear the sound
      sources.connect(audioCtx.destination);
      // start the source playing
      sources.start();

      // When the buffer source stops playing, disconnect everything

      sources.onended = function() {
        _this.isPlay = false;

        console.log("ended");

        sources.disconnect(audioCtx.destination);
      };

      _this.source = sources;
    }
  }
};
</script>

<style lang="scss">
.qj_audio {
  width: 400px;
  height: 44px;
  background: linear-gradient(
    90deg,
    rgba(255, 163, 201, 1),
    rgba(206, 185, 255, 1)
  );
  border-radius: 4px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin-top: 20px;
  cursor: pointer;

  span {
    font-size: 26px;
    color: #ffffff;
    margin-left: 15px;
  }
  span.voice-normal {
    width: 32px;
    height: 28px;
    background-image: url(../assets/images/bliiq@2x.png);
    background-repeat: no-repeat;
    background-size: 100% 100%;
  }
  span.voicePlay {
    background-image: url(../assets/images/bliiq_play.gif);
  }
}
</style>
