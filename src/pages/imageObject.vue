<template>
  <div>
    <v-layout column>
      <h2>画像のオブジェクト推定</h2>
      <br/>
      <input type="file" accept="image/*" @change="pickImage" />
      <br>
      <div v-if="result" style="color: orangered">
        これは、{{ result }}です。
      </div>
      <v-progress-circular
        v-else-if="loading"
        indeterminate
        color="primary"
      ></v-progress-circular>
      <div v-else>
        推定できません（ ｉ _ ｉ ）
      </div>
      <br>
    </v-layout>
    <div class="imageArea">
      <img :src="src">
    </div>
  </div>
</template>

<script>
import * as cocoSsd from '@tensorflow-models/coco-ssd';

export default {
  data () {
    return {
      src: '',
      result: '',
      loading: false
    }
  },

  methods: {
    pickImage(e) {
      this.loading = true;
      const reader = new FileReader();
      reader.onload = e => {
        this.src = e.target.result;
        this.estimateObject(e.target.result);
      };
      reader.readAsDataURL(e.target.files[0]);
    },

    async estimateObject(data) {
      const img = new Image();
      img.src = data;
      img.onload = async () => {
        const model = await cocoSsd.load();
        const predictions = await model.detect(img);
        this.result = !!predictions[0] ? predictions[0].class: '';
        this.loading = false;
      };
    }
  }
}

</script>

<style scoped>
  .imageArea {
    display: inline-block;
    min-width: 200px;
    min-height: 200px;
    border: 5px dashed #eee;
    padding: 10px;
  }
</style>
