<template>
  <div id="app">
    <CameraSelection v-on:change-camera="onChangeCamera"/>
    <QRCodeReader
      ref="qrCodeReader"
      :cameraType="cameraType"
      :cameraSize="cameraSize"
      v-on:onReadCode="onReadCode"
    />
    <div>{{ code }}</div>
  </div>
</template>

<script>
import CameraSelection from "./components/CameraSelection.vue";
import QRCodeReader from "./components/QRCodeReader.vue";

export default {
  name: "app",
  components: {
    CameraSelection,
    QRCodeReader
  },
  data() {
    return {
      cameraType: { type: "facingMode", device: "environment" },
      cameraSize: null,
      code: ""
    };
  },
  methods: {
    onChangeCamera(cameraType, cameraSize) {
      this.cameraType = cameraType;
      this.cameraSize = cameraSize;
      this.$nextTick(() => this.$refs.qrCodeReader.updateCamera());
    },
    onReadCode(data) {
      if (data) {
        console.log(data);
        this.code = data;
      }
    }
  }
};
</script>
