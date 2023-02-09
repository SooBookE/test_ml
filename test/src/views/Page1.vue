<template>
  <h1>1page[아버지와 자식 키 분석]</h1>
  <input type="file" name="" id="" v-on:change="readExcel" />
  <hr />
  <button v-on:click="analize" style="margin-right: 20px">분석시작</button>
  <button @click="predict">아버지키가 175일 때 결과 예측하기</button>
  <div id="result"></div>
</template>

<script>
/* eslint-disable */
export default {
  name: "app",
  data() {
    return {
      father_train: [],
      son_train: [],
      father_test: [],
      son_test: [],
      insert: 175,
      result: 0,
    };
  },
  methods: {
    readExcel(e) {
      const arr1 = [];
      const arr2 = [];
      const arr3 = [];
      const arr4 = [];
      let input = e.target;
      let reader = new FileReader();
      reader.onload = function () {
        let data = reader.result;
        let wordBook = XLSX.read(data, { type: "binary" });
        const x = wordBook.Sheets.train;

        for (let i = 2; i < 757; i++) {
          let line = [],
            line2 = [];
          let str = String.fromCharCode(65) + i;
          line.push(x[str].v);
          let str2 = String.fromCharCode(66) + i;
          line2.push(x[str2].v);

          arr1.push(line);
          arr2.push(line2);
        }

        for (let i = 2; i < 325; i++) {
          let line = [],
            line2 = [];
          let str = String.fromCharCode(65) + i;
          line.push(x[str].v);
          let str2 = String.fromCharCode(66) + i;
          line2.push(x[str2].v);

          arr3.push(line);
          arr4.push(line2);
        }
      };
      reader.readAsBinaryString(input.files[0]);
      this.father_train = arr1;
      this.son_train = arr2;
      this.father_test = arr3;
      this.son_test = arr4;
    },
    analize() {
      const x_train = tf.tensor(this.father_train);
      const y_train = tf.tensor(this.son_train);

      const X = tf.input({ shape: [1] });
      const Y = tf.layers.dense({ units: 50, activation: "relu" }).apply(X);
      const A = tf.layers.dense({ units: 50, activation: "relu" }).apply(Y);
      const B = tf.layers.dense({ units: 50, activation: "relu" }).apply(A);
      const C = tf.layers.dense({ units: 50, activation: "relu" }).apply(B);
      const Z = tf.layers.dense({ units: 1, activation: "sigmoid" }).apply(C);
      const model = tf.model({ inputs: X, outputs: Z });
      const compileParam = {
        optimizer: tf.train.adam(),
        loss: tf.losses.meanSquaredError,
      };
      model.compile(compileParam);

      const x_test = tf.tensor(this.father_test);
      const y_test = tf.tensor(this.son_test);

      tfvis.show.modelSummary(
        { name: "Model Architecture", tab: "Model" },
        model
      );
      model.collectedTrainableWeights.forEach((v) => console.log(v));
      // console.log(model);
      const metrics = ["loss", "val_loss", "acc", "val_acc"];
      const container = {
        name: "Model Training",
        tab: "Model",
        styles: { height: "1000px" },
      };
      const fitCallbacks = tfvis.show.fitCallbacks(container, metrics);

      const x_train_max = x_train.max();
      const x_train_min = x_train.min();
      const y_train_max = y_train.max();
      const y_train_min = y_train.min();

      const normTrain_x = x_train
        .sub(x_train_min)
        .div(x_train_max.sub(x_train_min));
      const normTrain_y = y_train
        .sub(y_train_min)
        .div(y_train_max.sub(y_train_min));

      const x_test_max = x_test.max();
      const x_test_min = x_test.min();
      const y_test_max = y_test.max();
      const y_test_min = y_test.min();

      const normTest_x = x_test.sub(x_test_min).div(x_test_max.sub(x_test_min));
      const normTest_y = y_test.sub(y_test_min).div(y_test_max.sub(y_test_min));

      const fitParam = {
        epochs: 10,
        validationData: [normTest_x, normTest_y],
        shuffle: true,
        callbacks: fitCallbacks,
      };

      model.fit(normTrain_x, normTrain_y, fitParam).then((_) => {
        const trans1 = this.father_test;
        const trans2 = this.son_test;
        (async function () {
          const result = await model.predict(normTest_x);
          // result.print();
          const real_result = await result
            .mul(x_test_max.sub(x_test_min))
            .add(x_test_min);
          // real_result.print();
          const result_arr = await real_result.arraySync();
          const original = [];
          const predicted = [];

          for (let i = 0; i < trans1.length; i++) {
            original[i] = await {
              x: trans1[i][0],
              y: trans2[i][0],
            };
            predicted[i] = await {
              x: trans1[i][0],
              y: result_arr[i][0],
            };
          }

          tfvis.render.scatterplot(
            { name: "Model Predictions vs Original Data" },
            {
              values: [original, predicted],
              series: ["original", "predicted"],
            },
            {
              xLabel: "Father's Height",
              yLabel: "Son's Height",
              height: 300,
              zoomToFit: true,
            }
          );
        })();
        model.save("localstorage://Training-Complete");
        console.log("학습 저장 완료.");
      });
    },
    predict() {
      let size;
      tf.loadLayersModel("localstorage://Training-Complete").then((model) => {
        size = [[this.insert]];
        const trans1 = this.father_test;
        (async function () {
          const x_test = await tf.tensor(trans1);
          const x_test_max = await x_test.max();
          const x_test_min = await x_test.min();

          const xt = await tf.tensor(size);
          const normTest_x = await xt
            .sub(x_test_min)
            .div(x_test_max.sub(x_test_min));

          const Result = await model.predict(normTest_x);
          const unnormResult = await Result.mul(x_test_max.sub(x_test_min)).add(
            x_test_min
          );
          const result = await unnormResult.arraySync();
          const div = document.getElementById("result");
          div.innerHTML = result[0][0].toFixed(1) + "cm";
        })();
      });
    },
  },
};
</script>

<style></style>
