<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="records"
      hide-default-footer
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>中信匯款</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <!-- <v-btn color="success" dark class="mb-2" @click="exportToTxt">
                Export
              </v-btn> -->
              <v-btn color="primary" dark class="mb-2" @click="importDialog = true">匯入<v-icon small> fas fa-upload </v-icon></v-btn>


              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                
                新增單筆
                <v-icon small> fas fa-plus-circle </v-icon>
              </v-btn>
            </template>

            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="editedItem.bankNo"
                        label="銀行代號"
                        maxlength="3"
                        clearable
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="editedItem.name"
                        label="戶名"
                        clearable
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="editedItem.acc"
                        label="帳號"
                        maxlength="16"
                        clearable
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="editedItem.money"
                        label="金額"
                        clearable
                        maxlength="14"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="editedItem.email"
                        label="E-mail"
                        clearable
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  取消
                </v-btn>
                <v-btn color="blue darken-1" text @click="save"> 儲存 </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline"
                >請問是否要移除?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >否</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >是</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">
          fas fa-edit
        </v-icon>
        <v-icon small @click="deleteItem(item)"> fas fa-trash </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize"> Reset </v-btn>
      </template>
    </v-data-table>
    <br>
    <v-btn color="success" @click="exportToTxt"  class="float-right">匯出(給Ecash)<v-icon small> fas fa-download </v-icon></v-btn>
    <a id="downloadAnchorElem" ref="downloadLink"></a>

    <br />
    
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
</template>

<script>
import AES from "crypto-js/aes";
import encUtf8 from "crypto-js/enc-utf8";

export default {
  name: "BatchFile",
  components: {},
  data() {
    return {
      headers: [
        {
          text: "銀行代號",
          value: "bankNo",
        },
        {
          text: "戶名",
          value: "name",
        },
        {
          text: "帳號",
          value: "acc",
        },
        {
          text: "金額",
          value: "money",
        },
        {
          text: "E-mail",
          value: "email",
        },
        { text: "操作", value: "actions", sortable: false },
      ],
      records: [],
      importDialog: false,
      importText: null,
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      editedItem: {
        bankNo: "822",
        name: "",
        acc: "",
        money: 0,
        email: "",
      },
      defaultItem: {
        bankNo: "822",
        name: "",
        acc: "",
        money: 0,
        email: "",
      },
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "新增匯款資料" : "編輯匯款資料";
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
    exportToTxt() {
      let dataUri = this._prepareContent();
      let downloadLink = this.$refs.downloadLink;
      downloadLink.setAttribute("href", dataUri);
      downloadLink.setAttribute(
        "download",
        "ecash_" + Date.now().toString() + ".txt"
      );
      downloadLink.click();
    },
    _prepareContent() {
      let fileContent = "";

      this.records.forEach((record) => {
        let row =
          record.bankNo +
          leftPad(record.acc, 16) +
          leftPad(record.money, 14) +
          "00" +
          record.email;
        fileContent += rightPad(row, 150) + "\r\n";
      });
      let dataUri =
        "data:text/csv;charset=utf-8," + encodeURIComponent(fileContent);
      return dataUri;
    },

    importFromText() {
      if (this.importText == null || this.importText.length == 0) {
        alert("請先輸入資料");
        return
      }
      let lines = this.importText.split("\n");
      this.records = [];
      lines.forEach((line) => {
        let record = {};
        record["bankNo"] = line.slice(0, 3);
        record["acc"] = line.slice(3, 19);
        record["money"] = parseInt(line.slice(19, 33));
        record["email"] = line.slice(35, -1).replace(/\s/g, "");
        record["name"] = "";
        this.records.push(record);
      });
      this.importDialog = false;
      alert("匯入成功");
    },

    saveToLocalStorage() {
      let encryptedStr = encrypt(JSON.stringify(this.records), "AAA");
      localStorage.setItem("records", encryptedStr);
    },

    loadFromLocalStorage() {
      const savedRecords = localStorage.getItem("records");
      if (savedRecords) {
        let decryptedStr = decrypt(savedRecords, "AAA");
        this.records = JSON.parse(decryptedStr);
      }
    },

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
  },
};

function encrypt(str, key) {
  return AES.encrypt(str, key).toString();
}

function decrypt(str, key) {
  let bytes = AES.decrypt(str, key);
  return bytes.toString(encUtf8);
}

function leftPad(str, max) {
  str = str.toString();
  return str.length < max ? leftPad("0" + str, max) : str;
}

function rightPad(str, max) {
  str = str.toString();
  return str.length < max ? rightPad(str + " ", max) : str;
}
</script>
