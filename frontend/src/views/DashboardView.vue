<template>
  <div class="container-fluid">
    <div class="row">
      <nav class="col-md-2 d-none d-md-block sidebar">
        <div class="sidebar-sticky">
          <h5 class="sidebar-heading">Beacon of Life Hospital</h5>
          <ul class="nav flex-column">
            <li class="nav-item" v-if="isAdmin">
              <router-link class="nav-link" to="/doctors">Doctor</router-link>
            </li>
            <li class="nav-item" v-if="isAdmin">
              <router-link class="nav-link" to="/patients">Patient</router-link>
            </li>
            <li class="nav-item" v-if="isAdmin">
              <router-link class="nav-link" to="/appointments">Appointment</router-link>
            </li>
            <li class="nav-item" v-if="isAdmin">
              <router-link class="nav-link" to="/medicalrecords">Medical Record</router-link>
            </li>
            <li class="nav-item" v-if="isAdmin">
              <router-link class="nav-link" to="/users">User Records</router-link>
            </li>
            <li class="nav-item" v-if="isDoctor">
              <router-link class="nav-link" to="/patients">My Patients</router-link>
            </li>
            <li class="nav-item" v-if="isDoctor || isPatient">
              <router-link class="nav-link" to="/appointments">My Appointments</router-link>
            </li>
            <li class="nav-item" v-if="isDoctor || isPatient">
              <router-link class="nav-link" to="/medicalrecords">My Medical Records</router-link>
            </li>
          </ul>
          <div class="sidebar-footer mt-4">
            <router-link class="btn btn-danger btn-sm" to="/logout">Logout</router-link>
            <button type="button" class="btn btn-primary btn-sm mt-2" @click="viewProfile">My Profile</button>
          </div>
        </div>
      </nav>
      <main role="main" class="col-md-9 ml-sm-auto col-lg-10 px-4">
        <router-view />
      </main>
    </div>

    <Modal v-if="showEditUserModal" @close="showEditUserModal = false">
      <template v-slot:header>
        <h5>Edit Profile Details</h5>
      </template>
      <template v-slot:body>
        <form @submit.prevent="updateUser" class="registration-form">
          <div class="form-group">
            <label for="name">Name:</label>
            <input class="form-control" type="text" v-model="editUserData.name" id="name" required>
          </div>
          <div class="form-group">
            <label for="email">Email:</label>
            <input class="form-control" type="email" v-model="editUserData.email" id="email" required>
          </div>
          <div class="form-group">
            <label for="password">Password:</label>
            <input class="form-control" type="password" v-model="editUserData.password" id="password" required>
          </div>
          <div class="form-group">
            <label for="userType">User Type:</label>
            <select class="form-control" id="userType" v-model="editUserData.userType" required>
              <option value="admin">Administrator</option>
              <option value="doctor">Doctor</option>
              <option value="patient">Patient</option>
            </select>
          </div>
          <button type="submit" class="btn btn-primary mt-2">Update</button>
        </form>
      </template>
    </Modal>
  </div>
</template>

<script>
import Modal from '@/components/PopUpModal.vue';
import axios from 'axios';

export default {
  name: 'DashBoard',
  components: {
    Modal
  },
  data() {
    return {
      user: null,
      showEditUserModal: false,
      editUserData: {
        id: '',
        name: '',
        email: '',
        password: '',
        userType: '',
        updated_at: ''
      }
    };
  },
  computed: {
    isAdmin() {
      return this.user && this.user.userType === 'admin';
    },
    isDoctor() {
      return this.user && this.user.userType === 'doctor';
    },
    isPatient() {
      return this.user && this.user.userType === 'patient';
    }
  },
  created() {
    this.loadUserFromLocalStorage();
  },
  methods: {
    loadUserFromLocalStorage() {
      const user = localStorage.getItem('user');
      if (user) {
        this.user = JSON.parse(user);
        this.editUserData = { ...this.user };
      }
    },
    viewProfile() {
      this.showEditUserModal = true;
    },
    async updateUser() {
      this.editUserData.updated_at = new Date().toISOString();
      await axios.put(`http://127.0.0.1:8000/api/doctor/${this.editUserData.id}`, this.editUserData, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer ' + localStorage.getItem('token')
        }
      });
      this.showEditUserModal = false;
    }
  }
};
</script>

<style scoped>
.sidebar {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  z-index: 100;
  padding: 48px 0 0;
  background-color: #2c3e50;
  color: #ecf0f1;
  transition: all 0.3s ease-in-out;
}

.sidebar:hover {
  background-color: #34495e;
}

.sidebar-sticky {
  position: relative;
  top: 0;
  height: 100%;
  padding-top: 1rem;
}

.sidebar-heading {
  padding: 1rem;
  font-size: 1.5rem;
  text-align: center;
  color: #f39c12;
  border-bottom: 1px solid #f39c12;
  margin-bottom: 1rem;
}

.nav-link {
  font-weight: 500;
  color: #ecf0f1;
  padding: 0.75rem 1rem;
  transition: color 0.3s ease, background-color 0.3s ease;
}

.nav-link:hover {
  color: #f39c12;
  background-color: #2c3e50;
}

.nav-link.active {
  color: #f39c12;
  background-color: #2c3e50;
}

.nav-item + .nav-item {
  border-top: 1px solid #34495e;
}

.main {
  margin-left: 200px;
  padding: 20px;
  background-color: #f8f9fa;
}

.sidebar-footer {
  text-align: center;
}

.btn-primary {
  background-color: #2980b9;
  border-color: #2980b9;
}

.btn-primary:hover {
  background-color: #3498db;
  border-color: #3498db;
}

.btn-danger {
  background-color: #e74c3c;
  border-color: #e74c3c;
}

.btn-danger:hover {
  background-color: #c0392b;
  border-color: #c0392b;
}

</style>
