<template>
  <v-container fluid>
    <div>
      <v-data-table
        :headers="headers"
        :items="records"
        hide-default-footer
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>薪資轉帳</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark class="mb-2" @click="importDialog = true">匯入<v-icon small> fas fa-upload </v-icon></v-btn>
                <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                  新增單筆
                  <v-icon small> fas fa-plus-circle </v-icon>
                </v-btn>
              </template>
              <v-card @keydown.enter.prevent="save">
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12">
                        <v-text-field
                          v-model="editedItem.name"
                          label="戶名"
                          clearable
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12">
                        <v-text-field
                          v-model="editedItem.accountNumber"
                          label="帳號"
                          maxlength="12"
                          clearable
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12">
                        <v-text-field
                          v-model="editedItem.amount"
                          label="金額"
                          type="number"
                          clearable
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12">
                        <v-text-field
                          v-model="editedItem.bankCode"
                          label="銀行代碼"
                          maxlength="4"
                          clearable
                        ></v-text-field>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">取消</v-btn>
                  <v-btn color="blue darken-1" text @click="save">儲存</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="headline">請問是否要移除?</v-card-title>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete">否</v-btn>
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm">是</v-btn>
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:body="{ items }">
          <draggable v-model="records" tag="tbody">
            <tr v-for="(item, index) in items" :key="index">
              <td>
                <v-icon small class="page-move">
                  fas fa-align-justify
                </v-icon>
              </td>
              <td>{{ item.name }}</td>
              <td>{{ item.accountNumber }}</td>
              <td>{{ item.amount }}</td>
              <td>{{ item.bankCode }}</td>
              <td>
                <v-icon small class="mr-2" @click="editItem(item)">
                  fas fa-edit
                </v-icon>
                <v-icon small @click="deleteItem(item)"> fas fa-trash </v-icon>
              </td>
            </tr>
          </draggable>
        </template>
      </v-data-table>
      <br />
      <v-btn color="success" @click="generateFile" class="float-right">
        匯出(給Ecash)
        <v-icon small>fas fa-download</v-icon>
      </v-btn>
      <v-dialog v-model="importDialog" width="85%">
        <v-card>
          <v-card-title primary-title> 匯入舊資料 </v-card-title>
          <v-card-text>
            <v-textarea
              outlined
              label="請手動貼上之前匯出的txt檔案內容"
              v-model="importText"
            ></v-textarea>
          </v-card-text>
        </v-card>
        <v-btn color="info" @click="importFromText">匯入</v-btn>
      </v-dialog>
    </div>
  </v-container>
</template>

<script>
import draggable from "vuedraggable";
import AES from "crypto-js/aes";
import encUtf8 from "crypto-js/enc-utf8";

export default {
  name: 'SalaryTransfer',
  components: { draggable },
  data() {
    return {
      dialog: false,
      dialogDelete: false,
      importDialog: false,
      importText: null,
      headers: [
        { text: "排序", value: "sort", sortable: false },
        { text: '戶名', value: 'name' },
        { text: '帳號', value: 'accountNumber' },
        { text: '金額', value: 'amount' },
        { text: '銀行代碼', value: 'bankCode' },
        { text: '操作', value: 'actions', sortable: false },
      ],
      records: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        accountNumber: '',
        amount: 0,
        bankCode: '822',
      },
      defaultItem: {
        name: '',
        accountNumber: '',
        amount: 0,
        bankCode: '822',
      },
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? '新增匯款資料' : '編輯匯款資料';
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
    records: {
      handler() {
        this.saveToLocalStorage();
      },
      deep: true,
    },
  },
  created() {
    this.loadFromLocalStorage();
  },
  methods: {
    editItem(item) {
      this.editedIndex = this.records.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
      this.editedIndex = this.records.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    deleteItemConfirm() {
      this.records.splice(this.editedIndex, 1);
      this.closeDelete();
    },
    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.records[this.editedIndex], this.editedItem);
      } else {
        this.records.push(this.editedItem);
      }
      this.close();
    },
    generateFile() {
      const header = 'H'.padEnd(500, ' ') + '\r\n';
      const content = this.records
        .map(record => {
          const recordType = '20000';
          const accountNumber = String(record.accountNumber || '').padEnd(12, ' ');
          const flag = '0';
          const amount = String((record.amount || 0) * 10).padStart(14, '0');
          const bankCode = String(record.bankCode || '').padStart(4, '0');
          
          let line = `${recordType}${accountNumber}${flag}${amount}${bankCode}`;
          return line.padEnd(500, ' ');
        })
        .join('\r\n');

      const fileContent = header + content;
      const blob = new Blob([fileContent], { type: 'text/plain;charset=big5' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      
      const today = new Date();
      const year = today.getFullYear() - 1911;
      const month = String(today.getMonth() + 1).padStart(2, '0');
      link.download = `salary_${year}${month}.txt`;
      
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    importFromText() {
      if (this.importText == null || this.importText.length == 0) {
        alert("請先輸入資料");
        return
      }
      let lines = this.importText.split(/\r?\n/).filter(line => line.trim().startsWith('20000'));
      this.records = [];
      lines.forEach((line) => {
        let record = {};
        record["accountNumber"] = line.slice(5, 17).trim();
        const amountStr = line.slice(18, 32);
        record["amount"] = parseInt(amountStr, 10) / 10;
        record["bankCode"] = line.slice(32, 36).trim();
        record["name"] = "";
        this.records.push(record);
      });
      this.importDialog = false;
      alert("匯入成功");
    },
    saveToLocalStorage() {
      let encryptedStr = encrypt(JSON.stringify(this.records), "SalaryTransferKey");
      localStorage.setItem("salaryRecords", encryptedStr);
    },
    loadFromLocalStorage() {
      const savedRecords = localStorage.getItem("salaryRecords");
      if (savedRecords) {
        let decryptedStr = decrypt(savedRecords, "SalaryTransferKey");
        this.records = JSON.parse(decryptedStr);
      }
    },
  },
};

function encrypt(str, key) {
  return AES.encrypt(str, key).toString();
}

function decrypt(str, key) {
  let bytes = AES.decrypt(str, key);
  return bytes.toString(encUtf8);
}
</script>

<style scoped>
.page-move {
  cursor: move;
}
</style>
