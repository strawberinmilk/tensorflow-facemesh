<template>
  <v-container>
    <video id="video"></video>
    <div id="face" :style="{ transform: `translate(${x}px, ${y}px)` }">顔だよ</div>
    <div id="leftEye">左目{{leftEye}}</div>
    <div id="rightEye">右目{{rightEye}}</div>
    <div id="mouth">口開度{{mouth}}</div>
  <!--
    <h1 id="status">すてーたす</h1>
    <h1 id="text">はろーわーるど</h1>
    -->
  </v-container>
</template>
<script src="./index.js"></script>

<script>
import * as facemesh from "@tensorflow-models/facemesh";
import * as tf from "@tensorflow/tfjs-core";
import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";
import delay from 'delay';

  export default {
    name: 'HelloWorld',

    data () {
      return {
        x: 0,
        y: 0,
        leftEye : 0,
        rightEye: 0,
        mouth: 0
        }
    },
    async mounted() {
      const videoDom = document.getElementById('video');
      navigator.mediaDevices.getUserMedia({
        video: true,
        audio: false,
      }).then(stream => {
        videoDom.srcObject = stream;
        videoDom.play();
      }).catch(e => {
        console.log(e);
      });

      await tf.setBackend('webgl');
      await tf.ready();

      await new Promise((resolve ,reject)=>{
        videoDom.addEventListener('loadeddata', (event) => {
          resolve();
        });
      })

      const model = await facemesh.load();
      const video = document.querySelector('video');

      console.log('stanby')

      const loop = async () => {
        const faces = await model.estimateFaces(video);
        console.log(faces.length)
        if (!faces.length) {
          await delay(100)
          loop()
          return
        };
        this.x = faces[0].boundingBox.topLeft[0]
        this.y = faces[0].boundingBox.topLeft[1]

        this.leftEye = Math.round(faces[0].mesh[159][1] - faces[0].mesh[145][1])
        this.rightEye = Math.round(faces[0].mesh[386][1] - faces[0].mesh[374][1])
        this.mouth = Math.round(faces[0].mesh[14][1] - faces[0].mesh[13][1])
        loop();
      }
      loop();
      /*
      setInterval(async () => {
        const faces = await model.estimateFaces(video);
        console.log(faces.length)
        if (!faces.length) {
          return
        };
        this.x = faces[0].boundingBox.topLeft[0]
        this.y = faces[0].boundingBox.topLeft[1]
      },4)
      */


      ///////////////////////////////////
      if ('webkitSpeechRecognition' in window) {
        console.log('いけるよ');
      } else {
        console.log('だめだよ');
      }
      navigator.mediaDevices.getUserMedia({audio: true, video: false})
        //SpeechRecognition = webkitSpeechRecognition || SpeechRecognition;
        const recognition = new webkitSpeechRecognition();
      
        recognition.onresult = (event) => {
          alert(event.results[0][0].transcript);
        }
      
        recognition.start();
    }
  }

</script>

<style>
#video{
	-ms-filter: blur(10px);
	filter: blur(10px);
}
</style>