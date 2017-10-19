<template>
  <div class="hello">
    <video id="local_video" autoplay playsinline style="width: 320px; height: 240px; border: 1px solid black;"></video>
    <div>
      <button @click="startVideo()">start</button>
      <input id="uploadFile" name="image" type="file" v-on:change="draw"/>
      <button id="capture" @click="captureImage()">Capture</button>
      <button @click="postImage()">post</button>
    </div>
    <canvas id="snapshot" width=320 height=240></canvas>
    <div>{{ responseData }}</div>
  </div>
</template>

<script>
import axios from 'axios'

const barcode_api_endpoint = 'https://tornado-barcode-reader.herokuapp.com/'

export default {
  name: 'hello',
  data: function () {
    return {
      msg: 'Welcome to Your Vue.js PWA',
      imageData:  null,
      responseData: 'hoge',
    }
  },
  methods : {
    startVideo: function () {
      navigator.getUserMedia = navigator.getUserMedia ||
        navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
      const constraints = {
        audio: false,
        video: true
      };
      // component内でエレメント取得するときはthis.$elを使う
      const video = this.$el.querySelector('video');
      const successCallback = function (stream) {
        window.stream = stream; // stream available to console
        if (window.URL) {
          alert('if')
          video.src = window.URL.createObjectURL(stream);
        } else {
          alert('else')
          video.srcObject = stream;
        }
      }

      const errorCallback = function (error) {
        console.log('navigator.getUserMedia error: ', error);
      }

      navigator.getUserMedia(constraints, successCallback, errorCallback)
    },
    captureImage: function () {
      const video = this.$el.querySelector('video')
      const snapshotCanvas = this.$el.querySelector('canvas');
      const context = snapshotCanvas.getContext('2d');
      // Draw the video frame to the canvas.
      context.drawImage(video, 0, 0, snapshotCanvas.width, snapshotCanvas.height);
      console.log(context.getImageData(0, 0, snapshotCanvas.width, snapshotCanvas.height))
    },
    postImage: function () {
      const snapshotCanvas = this.$el.querySelector('canvas');
      this.imageData = snapshotCanvas.toDataURL('image/png')

      const param = new FormData()
      param.append('upload_file', this.imageData.replace(/^.*,/, ''))
      const headers = {'Content-Type': 'multipart/form-data'}
      axios.post(barcode_api_endpoint, param, headers)
//      axios.post('http://localhost:5050', param, headers)
        .then( response => {
        console.log(response);
        this.responseData = response.data
      }).catch( error => {
        console.log(error);
      });
    },
    draw: function (evt) {
      console.log(evt)
      const file = evt.target.files[0];
      console.log(file)
      if (!file.type.match(/^image\/(png|jpeg|gif)$/)) return;

      const snapshotCanvas = this.$el.querySelector('canvas');
      const context = snapshotCanvas.getContext('2d');


      var image = new Image();
      var reader = new FileReader();

      reader.onload = function (e) {
        image.onload = function () {

          snapshotCanvas.width =  image.width
          snapshotCanvas.height = image.height

          context.drawImage(image, 0, 0); //canvasに画像を転写

        }


        image.src = e.target.result;
      }
      reader.readAsDataURL(file);

    }
  },
  mounted: function () {
    this.startVideo()
//    this.captureImage()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #35495E;
}
video {
  max-width: 100%;
  width: 320px;
}
canvas {
  border: solid 2px #003c75;
}
</style>
