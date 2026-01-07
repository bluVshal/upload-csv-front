<template>
  <LoadingSpinner 
    v-if="isSendingToApi || isUploadingFile" 
    :message="loadingMessage" 
  />
  <div>
    <header>
      <h1>CSV Reader</h1>
    </header>
  </div>
  <main>
    <div v-if="this.loading === 1">Loading data...</div>
    <div v-else-if="this.loading === 0">
      <div>
        <p> Please Upload CSV File</p><br />
        <label class="upload-button">
          Upload File
          <input type="file" @change="handleFileUpload" accept=".xlsx,.xls,.csv" hidden />
        </label>
      </div>
    </div>
    <div v-else-if="error" class="error">{{ this.error }}</div>
    <div v-else>
      <!-- Error Messages Section -->
      <div v-if="apiErrors && apiErrors.length > 0" class="error-section">
        <h3>Errors Found:</h3>
        <p v-for="(err, index) in apiErrors" :key="index" class="error-message">
          <strong>{{ err.first_name }} {{ err.last_name }}</strong>: {{ err.err_message || err.error }}
          <span v-if="err.err_code" class="error-code">(Error Code: {{ err.err_code }})</span>
        </p>
      </div>

      <div>
        <button class="action-button" @click="sendToAffApi">Add to Affiliates API</button>
        <button class="reset-button" @click="resetAll">Reset All</button>
      </div>
      <table>
        <thead>
          <tr>
            <th>Select</th>
            <th>Hostname</th>
            <th>First Name</th>
            <th>Last Name</th>
            <th>Phone Number</th>
            <th>Username</th>
            <th>Company</th>
          </tr>
        </thead>
        <tbody>
          <!-- Loop through the items array -->
          <tr v-for="item in this.responseData" :key="item.id">
            <td><input type="checkbox" :value="item" v-model="selectedItems" /></td>
            <td>{{ item.hostname }}</td>
            <td>{{ item.first_name }}</td>
            <td>{{ item.last_name }}</td>
            <td>{{ item.phone_number }}</td>
            <td>{{ item.username }}</td>
            <td>{{ item.company }}</td>
          </tr>
        </tbody>
      </table>
      <div>
        <button class="action-button" @click="sendToAffApi">Add to Affiliates API</button>
      </div>
    </div>
  </main>

  <RouterView />

</template>


<script>
import axios from 'axios';
import LoadingSpinner from './components/LoadingSpinner.vue';

export default {
  components: {
    LoadingSpinner
  },
  data() {
    return {
      selectedFile: null,
      loading: 0,
      responseData: [],
      error: '',
      sendToAPI: [],
      selectedItems: [],
      token: '',
      isSendingToApi: false,
      isUploadingFile: false,
      loadingMessage: 'Processing...',
      apiErrors: [],
    };
  },
  methods: {
    handleFileUpload(event) {
      const uploadedFile = event.target.files[0];
      if (uploadedFile) {
        this.selectedFile = uploadedFile; // Store the File object in the reactive variable
        this.uploadFile();
      }
    },

    resetAll(){
      window.location.reload();
    },

    uploadFile() {
      if (!this.selectedFile) return;

      // Show loading spinner
      this.isUploadingFile = true;
      this.loadingMessage = 'Uploading CSV file...';

      // Create a FormData object
      const formData = new FormData();
      // Append the file to the FormData object with a key (e.g., 'file')
      formData.append('file', this.selectedFile);

      // Send the POST request to your Node.js endpoint
      axios.post('https://upload-csv-back.onrender.com/api/upload', formData, {
        headers: {
          // This header is important for file uploads
          'Content-Type': 'multipart/form-data'
        }
      })
        .then(response => {
          this.isUploadingFile = false;
          this.loading = 1;
          this.responseData = response.data.data;
          this.responseData = [].concat(response.data.data);
          alert('File uploaded successfully!');
          this.loading = 2;
        })
        .catch(error => {
          this.isUploadingFile = false;
          console.error('Error uploading file:', error);
          alert('File upload failed.');
          this.error = error;
        });
    },

    /*  async executeRecaptcha() {
        return new Promise((resolve) => {
          window.grecaptcha.ready(async () => {
            const token = await window.grecaptcha.execute('6LeR49UfAAAAAO7J7vKDCD4vM-Lq5ZA_NsquS9J_', { action: 'submit' });
            console.log(token)
            resolve(token);
          });
        });
      },*/

    async sendToAffApi() {
      if (!this.selectedFile) return;

      // Get reCAPTCHA token
      /*const token = await this.executeRecaptcha();
      this.token = token;*/

      if (this.selectedItems && this.selectedItems.length > 0) {
        this.sendToAPI = [].concat(this.selectedItems);
      }
      else {
        this.sendToAPI = [].concat(this.responseData);
      }

      // Add token to the payload
      const payload = {
        data: this.sendToAPI,
        /* recaptchaToken: token*/
      };

      // Show loading spinner
      this.isSendingToApi = true;
      this.loadingMessage = `Sending ${this.sendToAPI.length} record(s) to API...`;

      axios.post('https://upload-csv-back.onrender.com/api/send-to-api', payload, {
        headers: {
          'Content-Type': 'application/json'
        }
      })
        .then(response => {
          this.isSendingToApi = false;
          const successCount = response.data.successCount || 0;
          const errorCount = response.data.errorCount || 0;
          
          // Store API errors for display
          this.apiErrors = response.data.errors || [];
          
          if (errorCount > 0) {
            alert(`API request completed!\nSuccess: ${successCount}\nFailed: ${errorCount}`);
          } else {
            alert(`All ${successCount} record(s) sent to Affiliates API successfully!`);
          }
          
          this.loading = 2;
        })
        .catch(error => {
          this.isSendingToApi = false;
          console.error('Error sending to API:', error);
          alert('Send to API failed.');
          this.error = error;
        });
    }
  }
};
</script>


<style scoped>
main {
  padding: 2rem;
}

h1 {
  padding: 1rem 0 0 1.8rem;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
  margin-bottom: 1.5rem;
  border: 1px ivory solid;
  border-radius: 1em;
  overflow: hidden;  
}

th,
td {
  padding: 0.75rem;
  text-align: left;
  color: rgb(3, 20, 255);
  background-color: #FFBF00;
  width: 9rem;
  border-bottom: 1px solid white; 
}

th {
  background-color: #FFFFE0;
  font-weight: bold;
  color: blue;
  width: 9rem;
}

tr:hover {
  background-color: #f9f9f9;
  color: green;
}

.upload-button,
.action-button {
  display: inline-block;
  padding: 10px 20px;
  background-color: #0e4500;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  text-transform: uppercase;
  transition: background-color 0.3s;
}

.reset-button {
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  text-transform: uppercase;
  transition: background-color 0.3s;
  display: inline-block;
  padding: 10px 20px;
  background-color: #1565c0;
  color: ivory;
  margin: 0 0 0 1rem;
}

.reset-button:hover {
  background-color: #156500;
}

.upload-button:hover,
.action-button:hover {
  background-color: #1565c0;
}

.upload-button {
  margin-bottom: 1rem;
}

.error {
  color: red;
  padding: 1rem;
}

.error-section {
  background-color: #fff3cd;
  border: 2px solid #ffc107;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1.5rem;
}

.error-section h3 {
  color: #856404;
  margin: 0 0 0.75rem 0;
  font-size: 1.1rem;
}

.error-message {
  color: #721c24;
  background-color: #f8d7da;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
  padding: 0.5rem 0.75rem;
  margin-bottom: 0.5rem;
  font-size: 0.95rem;
}

.error-message:last-child {
  margin-bottom: 0;
}

.error-code {
  color: #6c757d;
  font-size: 0.85rem;
  margin-left: 0.5rem;
}
</style>
