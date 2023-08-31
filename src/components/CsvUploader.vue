<template>
  <div class="container">
    <div class="upload-section">
      <label for="fileInput" class="upload-label">
        <input type="file" id="fileInput" ref="fileInput" @change="handleFileChange" class="file-input" />
        <span class="upload-icon">&#8681;</span> Importar CSV
      </label>
      <span v-if="selectedFile" class="file-selected">
        Archivo seleccionado: {{ selectedFile.name }}
      </span>
      <button @click="uploadCsv" :disabled="uploading || loading || !selectedFile" class="upload-button">
        <span v-if="uploading">Subiendo...</span>
        <span v-else>Subir CSV</span>
      </button>
      <div v-if="loading" class="spinner"></div>

    </div>
    <ul v-if="items.length > 0" class="item-list">
      <li v-for="item in items" :key="item.id" class="item">
        <div class="item-details">
          <p class="item-name">{{ item.name }} <span class="item-contact">{{ item.email }} | {{ item.phone }}</span> </p>

        </div>
      </li>
    </ul>
  </div>
</template>

<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.file-selected {
  margin-left: 10px;
  color: #555;
}

.upload-section {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.upload-label {
  background-color: #3498db;
  color: #fff;
  border-radius: 4px;
  padding: 10px 20px;
  cursor: pointer;
  display: inline-block;
  transition: background-color 0.3s ease-in-out;
}

.upload-label:hover {
  background-color: #2980b9;
}

.file-input {
  display: none;
}

.upload-icon {
  font-size: 16px;
  margin-right: 8px;
}

.upload-button {
  background-color: #3498db;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  margin-left: 10px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease-in-out;
}

.upload-button:hover {
  background-color: #2980b9;
}

.upload-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left: 4px solid #3498db;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.item-list {
  list-style: none;
  padding: 0;
}

.item {
  border: 1px solid #ccc;
  margin: 10px 0;
  border-radius: 4px;
  display: flex;
  align-items: center;
}

.item-details {
  flex: 1;
}

.item-name {
  font-size: 18px;
  font-weight: bold;
}

.item-contact {
  font-size: 14px;
  color: #555;
  font-weight: 100;
}

.error-message {
  background-color: #f44336;
  color: white;
  text-align: center;
  padding: 10px;
  border-radius: 4px;
  margin-top: 20px;
}
</style>



<script>
export default {
  data() {
    return {
      selectedFile: null,
      items: [],
      loading: false,
      uploading: false,
      count: 0,
    };
  },
  methods: {
    async handleFileChange(event) {
      this.selectedFile = event.target.files[0];
    },
    async uploadCsv() {
      this.uploading = true;
      try {
        const fileContent = await this.readFileContent(this.selectedFile);
        const records = this.parseCsvContent(fileContent);

        for (const record of records) {
          const formattedRecord = this.formatRecord(record);
          await this.uploadRecord(formattedRecord);
        }

        await this.fetchItems();
      } catch (error) {
        console.log(error);
      } finally {
        this.uploading = false;
      }
    },
    async readFileContent(file) {
      return new Promise((resolve, reject) => {
        const fileReader = new FileReader();
        fileReader.onload = (event) => resolve(event.target.result);
        fileReader.onerror = (error) => reject(error);
        fileReader.readAsText(file);
      });
    },
    parseCsvContent(content) {
      return content.split("\n").map((line) => line.trim().split(","));
    },
    formatRecord(record) {
      const [name, phone, email] = record;
      const formattedPhone = this.formatPhoneNumber(phone);
      return { name, phone: formattedPhone, email };
    },
    async uploadRecord(record) {
      try {
        await fetch(
          "https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/items",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(record),
          }
        );

      } catch (error) {
        console.error(error);
      }
    },
    async fetchItems() {
      try {
        const response = await fetch(
          "https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/items"
        );
        const data = await response.json();
        this.items = data.items;
      } catch (error) {
        console.error("Error:", error);
      }
    },
    formatPhoneNumber(phone) {
      const cleaned = ("" + phone).replace(/\D/g, "");
      if (cleaned.length === 10) {
        const match = cleaned.match(/^(\d{3})(\d{3})(\d{4})$/);
        if (match) {
          return `${match[1]}-${match[2]}-${match[3]}`;
        }
      } else if (cleaned.length === 9) {
        return `${cleaned.slice(0, 3)}-${cleaned.slice(3, 6)}-${cleaned.slice(6)}0`;
      }
      return phone;
    },
  },
  mounted() {
    this.fetchItems();
  },
};
</script>