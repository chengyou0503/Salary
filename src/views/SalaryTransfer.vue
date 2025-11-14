<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <h1 class="text-h4">薪資轉帳</h1>
      </v-col>
    </v-row>
    <v-data-table
      :headers="headers"
      :items="records"
      hide-default-footer
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>薪資資料</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                新增資料
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
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">fas fa-edit</v-icon>
        <v-icon small @click="deleteItem(item)">fas fa-trash</v-icon>
      </template>
    </v-data-table>
    <br />
    <v-btn color="success" @click="generateFile" class="float-right">
      產生薪資轉帳檔
      <v-icon small>fas fa-download</v-icon>
    </v-btn>
  </v-container>
</template>

<script>
export default {
  name: 'SalaryTransfer',
  data() {
    return {
      dialog: false,
      dialogDelete: false,
      headers: [
        { text: '帳號', value: 'accountNumber' },
        { text: '金額', value: 'amount' },
        { text: '銀行代碼', value: 'bankCode' },
        { text: '操作', value: 'actions', sortable: false },
      ],
      records: [],
      editedIndex: -1,
      editedItem: {
        accountNumber: '',
        amount: 0,
        bankCode: '',
      },
      defaultItem: {
        accountNumber: '',
        amount: 0,
        bankCode: '',
      },
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? '新增資料' : '編輯資料';
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
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
      let header = 'H'.padEnd(500, ' ') + '\n';
      let content = this.records
        .map(record => {
          const recordType = '20000';
          const accountNumber = String(record.accountNumber).padEnd(12, ' ');
          const unknownField = '0'; // As per example
          const amount = String(record.amount).padStart(7, '0');
          const bankCode = String(record.bankCode).padEnd(4, ' ');
          let line = `${recordType}${accountNumber}${unknownField}00000${amount}${bankCode}`;
          return line.padEnd(500, ' ');
        })
        .join('\n');

      const fileContent = header + content;
      const blob = new Blob([fileContent], { type: 'text/plain;charset=big5' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = 'salary_transfer.txt';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
};
</script>

<style scoped>
/* 在這裡添加組件特有的樣式 */
</style>
