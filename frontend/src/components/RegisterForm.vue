<template>
  <div class="registration-container">
    <h1>Beacon of Life Hospital</h1>
    <form @submit.prevent="register" class="registration-form">
      <div class="form-group">
        <label for="name">Name:</label>
        <input type="text" v-model="name" id="name" required>
      </div>
      <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" v-model="email" id="email" required>
      </div>
      <div class="form-group">
        <label for="password">Password:</label>
        <input type="password" v-model="password" id="password" required>
      </div>
      <div class="form-group">
        <label for="userType">User Type:</label>
        <select id="userType" v-model="userType" required>
          <option value="admin">Administrator</option>
          <option value="doctor">Doctor</option>
          <option value="patient">Patient</option>
        </select>
      </div>
      <button type="submit">Register</button>
    </form>
    <div v-if="error" class="error">{{ error }}</div>
    <span>Already have an account? <router-link to="/login" class="login-link">Login here</router-link></span>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      name: '',
      email: '',
      password: '',
      userType: '',
      error: ''
    };
  },
  methods: {
    async register() {
      try {
        const response = await axios.post('http://127.0.0.1:8000/api/register', {
          name: this.name,
          email: this.email,
          password: this.password,
          userType: this.userType
        });
        console.log(response.data);

        // Show success alert
        alert("Registration successful!");

        // Clear form
        this.name = '';
        this.email = '';
        this.password = '';
        this.userType = '';
        this.error = '';
      } catch (err) {
        if (err.response) {
          this.error = `Error: ${err.response.data.message}`;
          console.error(err.response.data);
        } else if (err.request) {
          this.error = 'No response from server. Please try again later.';
          console.error(err.request);
        } else {
          this.error = 'Request error. Please check your input and try again.';
          console.error('Error', err.message);
        }
      }
    }
  }
};
</script>

<style>
.registration-container {
  max-width: 500px;
  margin: 50px auto;
  padding: 30px;
  border: 1px solid #e0e0e0;
  border-radius: 15px;
  background-color: #f5f5f5;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
  font-family: 'Arial', sans-serif;
  transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
}

.registration-container:hover {
  transform: translateY(-10px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

h1 {
  color: #2c3e50;
  margin-bottom: 20px;
  font-size: 26px;
  font-weight: bold;
}

.registration-form {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.form-group {
  margin-bottom: 15px;
  width: 100%;
  text-align: left;
}

label {
  display: block;
  margin-bottom: 5px;
  color: #34495e;
  font-weight: bold;
}

input[type="text"],
input[type="email"],
input[type="password"],
select {
  width: 100%;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}

input[type="text"]:focus,
input[type="email"]:focus,
input[type="password"]:focus,
select:focus {
  border-color: #2980b9;
  outline: none;
}

button {
  padding: 12px 20px;
  background-color: #2980b9;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  margin-top: 10px;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

button:hover {
  background-color: #1abc9c;
  transform: translateY(-2px);
}

.error {
  color: red;
  margin-top: 10px;
  font-weight: bold;
  text-align: center;
}

.login-link {
  margin-top: 15px;
  color: #2980b9;
  text-decoration: none;
  font-weight: bold;
  transition: color 0.3s ease;
}

.login-link:hover {
  color: #1abc9c;
  text-decoration: underline;
}
</style>
