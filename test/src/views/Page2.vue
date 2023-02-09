<template>
  <h1>2page[Mobilenet 분석-Cam]</h1>
  <div>
    <div id="display"></div>
    <button @click="main()">스크린샷 분석</button>
    <button @click="alog()">디버거</button>
    <div>{{ result }}</div>
  </div>
</template>

<script>
const VideoElement = document.createElement("video"); // 캠 태그 생성.
export default {
  name: "p2",
  data() {
    return {
      result: "스크린샷 분석을 시작하세요.",
      output: "",
    };
  },
  methods: {
    alog() {
      console.log(tf);
    },
    async main() {
      this.result = "이미지 분석중...";
      const display = document.getElementById("display");

      display.appendChild(VideoElement);
      VideoElement.width = 400;
      VideoElement.height = 400;

      const cam = await tf.data.webcam(VideoElement);
      const net = await mobilenet.load();
      const img = await cam.capture();

      const cresult = await net.classify(img);

      this.output = `${cresult[0].className}(${(
        cresult[0].probability * 100
      ).toFixed(2)}%)`;
      this.result = "분석 결과 : " + this.output;

      img.print();
      tf.dispose(img);
    },
  },
};
</script>

<style></style>
