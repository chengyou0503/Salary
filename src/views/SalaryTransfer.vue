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
    <v-row v-if="parsedData.length > 0">
      <v-col cols="12">
        <v-btn color="primary" @click="generateFile">產生薪資轉帳檔</v-btn>
      </v-col>
    </v-row>
    <v-row v-if="parsedData.length > 0">
      <v-col cols="12">
        <v-card>
          <v-card-title>解析後資料</v-card-title>
          <v-card-text>
            <v-data-table
              :headers="headers"
              :items="parsedData"
              class="elevation-1"
            ></v-data-table>
          </v-card-text>
        </v-card>
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
      parsedData: [],
      headers: [
        { text: '帳號', value: 'accountNumber' },
        { text: '金額', value: 'amount' },
        { text: '銀行代碼', value: 'bankCode' },
      ],
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (e) => {
        this.fileContent = e.target.result;
        this.parseFileContent(this.fileContent);
      };
      reader.readAsText(file, 'big5'); // 假設檔案編碼為 big5
    },
    parseFileContent(content) {
      const lines = content.split('\n').filter(line => line.trim().length > 0);
      this.parsedData = lines.slice(1).map(line => {
        if (line.startsWith('20000')) {
          const accountNumber = line.substring(5, 17);
          const amount = parseInt(line.substring(18, 30), 10);
          const bankCode = line.substring(30, 35);
          return { accountNumber, amount, bankCode };
        }
        return null;
      }).filter(item => item !== null);
    },
    generateFile() {
      let output = 'H\n'; // Header line

      this.parsedData.forEach(row => {
        const recordType = '20000';
        const accountNumber = row.accountNumber.padEnd(12, ' ');
        const flag = '0';
        const amount = String(row.amount).padStart(12, '0');
        const bankCode = row.bankCode.padEnd(5, ' ');
        
        let line = recordType + accountNumber + flag + amount + bankCode;
        line = line.padEnd(500, ' '); // Pad the line to the required length
        output += line + '\n';
      });

      const blob = new Blob([output], { type: 'text/plain;charset=big5' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = 'salary_transfer_export.txt';
      link.click();
      URL.revokeObjectURL(link.href);
    },
  },
};
</script>

<style scoped>
/* 在這裡添加組件特有的樣式 */
</style>
