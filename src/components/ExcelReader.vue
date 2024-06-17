<template>
  <div>
    <input type="file" @change="handleFileUpload" />
    <div v-if="combinedData.length">
      <table class="excel-table">
        <thead>
          <tr>
            <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in combinedData" :key="rowIndex">
            <td
              v-for="(cell, cellIndex) in row"
              :key="cellIndex"
              :class="{'highlight': isTeamHeaderAndEmptyCell(headers[cellIndex], cell)}"
            >
              <textarea
                v-model="combinedData[rowIndex][cellIndex]"
                rows="1"
                cols="20"
                @mousedown.stop
                @mousemove.stop
              ></textarea>
              <div v-if="isTeamHeaderAndEmptyCell(headers[cellIndex], cell)" class="error-message">
                null값을 입력할 수 없습니다
              </div>
            </td>
          </tr>
        </tbody>
      </table>
      <button @click="saveDataToBackend">Save Data to Backend</button>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";

export default {
  data() {
    return {
      headers: [],
      combinedData: [],
    //  originalWorkbook: null,
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });

        const sheetNames = workbook.SheetNames;
        let headers = [];
        let combinedData = [];

        sheetNames.forEach((sheetName, index) => {
          const worksheet = workbook.Sheets[sheetName];
          const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

          if (jsonData.length > 0) {
            if (index === 0) {
              headers = jsonData[0];
            }
            combinedData = combinedData.concat(jsonData.slice(1));
          }
        });

        // Save the original workbook for later use
        this.originalWorkbook = workbook;

        this.headers = headers;
        this.combinedData = combinedData;
      };

      reader.readAsArrayBuffer(file);
    },
    saveDataToBackend() {
      if (!this.originalWorkbook) {
        console.error("No original workbook data found.");
        return;
      }

      // Convert modified data to JSON format
      const jsonData = this.convertToJSON(this.headers, this.combinedData);

      // Send JSON data to backend
      this.sendDataToBackend(jsonData);
    },
    convertToJSON(headers, data) {
      return data.map(row => {
        const obj = {};
        headers.forEach((header, index) => {
          obj[header] = row[index];
        });
        return obj;
      });
    },
    sendDataToBackend(jsonData) {
       // Check for empty cells in 'Team' header and prevent sending if found
      let hasEmptyCells = false;
      this.combinedData.forEach(row => {
        row.forEach((cell, index) => {
          if (this.isTeamHeaderAndEmptyCell(this.headers[index], cell)) {
            hasEmptyCells = true;
            console.log("ddddd");
          }
        });
      });

      if (hasEmptyCells) {
        console.error("Cannot save: Team 열에 비어 있는 셀이 있습니다.");
        console.log(hasEmptyCells);
        //return;
      } else {
        console.log(jsonData);
        console.log(hasEmptyCells);
      }

      // Send JSON data to backend
      //this.sendDataToBackend(jsonData);
     
    },
    isTeamHeaderAndEmptyCell(header, cell) {
      return header === 'Team' && (!cell || cell.trim() === '');
    },
  },
};
</script>

<style scoped>
.excel-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.excel-table th,
.excel-table td {
  border: 1px solid black;
  padding: 8px;
  text-align: left;
  vertical-align: top;
  word-wrap: break-word;
  white-space: pre-wrap;
  position: relative;
  overflow: hidden;
}

textarea {
  width: calc(100%);
  height: calc(100%);
  border: none;
  outline: none;
  padding: 8px;
  box-sizing: border-box;
  resize: none;
  overflow: auto;
  white-space: pre-wrap;
}

.highlight {
  position: relative;
}

.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
  font-weight: bold;
}
</style>
