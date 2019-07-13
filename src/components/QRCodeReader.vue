<template>
  <canvas/>
</template>

<script>
import "webrtc-adapter";
import jsQR from "jsqr";

export default {
  name: "QRCodeReader",
  props: {
    cameraType: Object,
    cameraSize: Object
  },
  mounted() {
    this.video = document.createElement("video");
    this.canvas = this.$el;
    this.ctx = this.canvas.getContext("2d");
    this.updateCamera();
  },
  methods: {
    _getConstraints() {
      const constraints = {};
      if (this.cameraType) {
        constraints[this.cameraType.type] = this.cameraType.device;
      }
      if (this.cameraSize) {
        constraints.width = { exact: this.cameraSize.width };
        constraints.height = { exact: this.cameraSize.height };
      }
      return constraints;
    },
    updateCamera() {
      if (this.stream) {
        this.stream.getTracks().forEach(track => {
          track.stop();
        });
      }

      navigator.mediaDevices
        .getUserMedia({ video: this._getConstraints() })
        .then(mediaStream => {
          this.stream = mediaStream;
          this.video.srcObject = mediaStream;
          // required to tell iOS safari we don't want fullscreen
          this.video.setAttribute("playsinline", true);
          this.video.play();
          requestAnimationFrame(this._tick);
        })
        .catch(error => {
          console.error(error);
          alert(error);
        });
    },
    _tick() {
      if (this.video.readyState === this.video.HAVE_ENOUGH_DATA) {
        this.canvas.height = this.video.videoHeight;
        this.canvas.width = this.video.videoWidth;
        this.ctx.drawImage(
          this.video,
          0,
          0,
          this.canvas.width,
          this.canvas.height
        );
        const imageData = this.ctx.getImageData(
          0,
          0,
          this.canvas.width,
          this.canvas.height
        );
        const code = jsQR(imageData.data, imageData.width, imageData.height, {
          inversionAttempts: "dontInvert"
        });
        if (code) {
          this._drawLine(
            code.location.topLeftCorner,
            code.location.topRightCorner
          );
          this._drawLine(
            code.location.topRightCorner,
            code.location.bottomRightCorner
          );
          this._drawLine(
            code.location.bottomRightCorner,
            code.location.bottomLeftCorner
          );
          this._drawLine(
            code.location.bottomLeftCorner,
            code.location.topLeftCorner
          );
          this.$emit("onReadCode", code.data);
        } else {
          this.$emit("onReadCode", "");
        }
      }
      requestAnimationFrame(this._tick);
    },
    _drawLine(begin, end) {
      this.ctx.beginPath();
      this.ctx.moveTo(begin.x, begin.y);
      this.ctx.lineTo(end.x, end.y);
      this.ctx.lineWidth = 4;
      this.ctx.strokeStyle = "#ff0000";
      this.ctx.stroke();
    }
  }
};
</script>
