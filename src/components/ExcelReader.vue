<template>
    <div>
      <input type="file" @change="handleFileUpload" />
      <div v-if="combinedData.length">
        <table>
          <thead>
            <tr>
              <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, rowIndex) in combinedData" :key="rowIndex">
              <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script>
  import * as XLSX from 'xlsx';
  
  export default {
    data() {
      return {
        headers: [],
        combinedData: [],
      };
    },
    methods: {
      handleFileUpload(event) {
        const file = event.target.files[0];
        const reader = new FileReader();
  
        reader.onload = (e) => {
          const data = new Uint8Array(e.target.result);
          const workbook = XLSX.read(data, { type: 'array' });
  
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
  
          this.headers = headers;
          this.combinedData = combinedData;
        };
  
        reader.readAsArrayBuffer(file);
      },
    },
  };
  </script>
  
  <style scoped>
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  th, td {
    border: 1px solid black;
    padding: 8px;
    text-align: left;
  }
  </style>
  