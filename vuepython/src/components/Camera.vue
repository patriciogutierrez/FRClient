<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6">
        <code v-if="device">{{ device.label }}</code>
        <div class="border">
          <vue-web-cam
            ref="webcam"
            :device-id="deviceId"
            width="100%"
            @started="onStarted"
            @stopped="onStopped"
            @error="onError"
            @cameras="onCameras"
            @camera-change="onCameraChange"
            style="transform: scaleX(-1);"
          />
        </div>

        <div class="row">
          <div class="col-md-12">
            <select v-model="camera">
              <!-- <option>-- Seleccionar --</option> -->
              <option
                v-for="device in devices"
                :key="device.deviceId"
                :value="device.deviceId"
              >{{ device.label }}</option>
            </select>
          </div>
          <div class="col-md-12">
            <button type="button" class="btn btn-primary" @click="onCapture">Tomar Foto</button>
            <!-- <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
            <button type="button" class="btn btn-success" @click="onStart">Start Camera</button> -->
          </div>
        </div>
      </div>
      <div class="col-md-6">
        <!-- <h2>Captured Image</h2> -->
        <br>
        <div class="border">
        <figure class="figure">
          <img :src="img" class="img-responsive" style="width:100%" />
        </figure>
        </div>
        <div class="row">
          <div class="col-md-12">
            <h2>{{respuesta}}</h2>
            <input v-model="sujeto" type="text" />
            <button type="button" class="btn btn-primary" @click="saveFace">Guardar</button>
            <button type="button" class="btn btn-primary" @click="sendData">Comparar</button>
            <!-- <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
            <button type="button" class="btn btn-success" @click="onStart">Start Camera</button> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { WebCam } from "vue-web-cam";
import axios from 'axios';
export default {
  name: "Camera",
  components: {
    "vue-web-cam": WebCam
  },
  data() {
    return {
      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      respuesta: null,
      sujeto : null
    };
  },
  computed: {
    device: function() {
      return this.devices.find(n => n.deviceId === this.deviceId);
    }
  },
  watch: {
    camera: function(id) {
      this.deviceId = id;
    },
    devices: function() {
      const [first, ...tail] = this.devices;
      tail ? null : null
      if (first) {
        this.camera = first.deviceId;
        this.deviceId = first.deviceId;
      }
    }
  },
  methods: {
    saveFace(){
      axios.post('http://104.237.150.159:7923/upload',{
        file : this.img.split(',')[1],
        usuario : this.sujeto
      }).then(res=>{
        this.respuesta = res.data
      })
    },
    sendData(){
      this.respuesta = ""
      axios.post('http://104.237.150.159:7923/predictImage',{
        file : this.img.split(',')[1],
        usuario : this.sujeto
      }).then(res=>{
        this.respuesta = res.data
      })
    },
    onCapture() {
      this.img = this.$refs.webcam.capture();
    },
    onStarted(stream) {
      console.log("On Started Event", stream);
    },
    onStopped(stream) {
      console.log("On Stopped Event", stream);
    },
    onStop() {
      this.$refs.webcam.stop();
    },
    onStart() {
      this.$refs.webcam.start();
    },
    onError(error) {
      console.log("On Error Event", error);
    },
    onCameras(cameras) {
      this.devices = cameras;
      console.log("On Cameras Event", cameras);
    },
    onCameraChange(deviceId) {
      this.deviceId = deviceId;
      this.camera = deviceId;
      console.log("On Camera Change Event", deviceId);
    }
  }
};
</script>