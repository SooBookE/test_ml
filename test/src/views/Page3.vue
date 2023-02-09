<template>
  <h1>3page[Mobilenet 분석-Img]</h1>
  <div>
    <div>
      <input
        ref="image"
        type="file"
        name="img"
        accept="image/*"
        multiple="multiple"
        id="input"
        @change="upIMG()"
      />
    </div>
    <img ref="img" v-if="imglink" :src="imglink" alt="사진 준비 중.." />
    <hr />
    <button @click="predict()">분석</button>
    <div>{{ result }}</div>
    <div id="result1">{{ result1 }}</div>
  </div>
</template>

<script>
export default {
  name: "p3",
  data() {
    return {
      imglink: "",
      result: "분석 버튼을 눌러 분석을 시작하세요.",
      result1: "정확도가 90%이상이면 이곳에 base64 코드가 출력됩니다.",
      inData: "",
      percentage: 0,
    };
  },
  methods: {
    upIMG() {
      const image = this.$refs.image.files[0];
      const reader = new FileReader();
      reader.readAsDataURL(image);
      reader.onload = (e) => {
        this.imglink = e.target.result;
      };
    },
    async predict() {
      const img = this.$refs.img;
      this.result = "이미지 분석 중...";
      this.trans = "번역 중...";

      await mobilenet
        .load()
        .then((model) => model.classify(img))
        .then((rst) => {
          this.percentage = (rst[0].probability * 100).toFixed(2);
          this.inData = `${rst[0].className}(${this.percentage}%)`;
          this.result = "분석 결과 : " + this.inData;
        });
      if (this.percentage >= 90) {
        this.result1 = this.imglink;
      } else {
        this.result1 =
          this.percentage + "%이므로, base64 코드를 출력하지 않았습니다.";
      }
      const tfimg = await tf.browser.fromPixels(img);
      tfimg.print();
      tf.dispose(tfimg);
    },
  },
};
</script>
<style>
#result1 {
  width: 95%;
  text-align: center;
  height: 80px;
  overflow: auto;
  word-break: break-all;
  margin: 10px;
}
</style>
