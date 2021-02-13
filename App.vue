<template>
  <v-app>
    <div>
      <v-card>
        <v-card-text>
          <v-file-input
            accept=".xlsx"
            hint="Please Confirm if you Select only .xlsx or .csv Formats"
            dense
            outlined
            placeholder="Attach Your File Here"
            small-chips
            v-model="files"
            multiple
            :show-size="true"
            v-on:change="File_To_Json($event)"
          >
          </v-file-input>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="dialog = !dialog">
            Cancel
          </v-btn>
          <v-btn color="secondary" @click="itemapicall()"
            ><v-icon class="ma-1">mdi-cloud-upload</v-icon> Upload
          </v-btn>
        </v-card-actions>
      </v-card>

      <v-main>
        <HelloWorld />
      </v-main>
    </div>
  </v-app>
</template>
<script>
import HelloWorld from "./components/HelloWorld";
import XLSX from "xlsx";
let filejs = new Array();
let WorkSheet;

export default {
  name: "App",

  components: {
    HelloWorld
  },

  data() {
    return {
      items: [],
      total: 0,
      files: [],
      date2: "2021-02-11",
      tp: 30,
      filename: "",
      filetojson: []
    };
  },
  methods: {
    File_To_Json(e) {
      if (e instanceof Array && e.length == 0) {
        console.log("inside if");
      } else {
        var files = e;
        var f = files[0];
        var reader = new FileReader();
        reader.onload = function(e) {
          var data = new Uint8Array(e.target.result);
          var workbook = XLSX.read(data, {
            type: "array",
            cellDates: true,
            dateNF: "MM/DD/YYYY"
          });
          let sheetName = workbook.SheetNames[0];
          let worksheet = workbook.Sheets[sheetName];
          WorkSheet = worksheet;
          console.log("File to json");
          filejs = XLSX.utils.sheet_to_json(worksheet);
          console.log(JSON.parse(JSON.stringify(filejs)));
        };
        reader.readAsArrayBuffer(f);
      }
    },
    itemapicall() {
      var sheetData = WorkSheet;
      console.log(sheetData);
      this.filename =
        "itemprocesslog" + new Date().toISOString().slice(0, 19) + ".csv";
      this.filetojson = JSON.parse(JSON.stringify(filejs));

      //console.log(this.filetojson[0].Date);

      console.log(this.filetojson.length);

      for (var i = 0; i < this.filetojson.length; i++) {
        var date1 = this.filetojson[i].Date;
        this.filename = date1.slice(0, 10);
        //console.log(this.filename);

        if (this.filename == this.date2) {
          console.log("inside date loop");
          for (var j = i; j > i - this.tp; j--) {
            this.total =
              this.total +
              (this.filetojson[j].Open +
                this.filetojson[j].High +
                this.filetojson[j].Low +
                this.filetojson[j].Close);
          }
          console.log(this.total / 30);
        }
      }
    }
  }
};
</script>

<style></style>
