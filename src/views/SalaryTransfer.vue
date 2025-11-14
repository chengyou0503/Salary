<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <h1 class="text-h4">薪資轉帳</h1>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-file-input
          label="選擇薪資檔案 (.txt)"
          accept=".txt"
          @change="handleFileUpload"
          show-size
          counter
        ></v-file-input>
      </v-col>
    </v-row>
    <v-row v-if="fileContent">
      <v-col cols="12">
        <v-card>
          <v-card-title>檔案內容</v-card-title>
          <v-card-text>
            <pre style="white-space: pre-wrap; word-wrap: break-word;">{{ fileContent }}</pre>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'SalaryTransfer',
  data() {
    return {
      fileContent: null,
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (e) => {
        this.fileContent = e.target.result;
      };
      reader.readAsText(file, 'big5'); // 假設檔案編碼為 big5
    },
  },
};
</script>

<style scoped>
/* 在這裡添加組件特有的樣式 */
</style>
