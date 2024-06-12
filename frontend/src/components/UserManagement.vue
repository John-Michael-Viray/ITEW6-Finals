<template>
    <DashboardView />
    <div class="hello">
        <h1>User Management</h1>
        <button class="btn btn-info btn-sm m-1" @click="addDoctor()">Add a User</button>
        <hr>
        <div class="table-container">
            <table class="table table-striped table-bordered">
                <thead class="thead-dark">
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Email</th>
                        <th>User Type</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="user in users" :key="user.id">
                        <td>{{ user.id }}</td>
                        <td>{{ user.name }}</td>
                        <td>{{ user.email }}</td>
                        <td>{{ user.userType }}</td>
                        <td>
                            <button class="btn btn-warning btn-sm m-1" @click="editUser(user)">Edit</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

    <Modal v-if="showAddUserModal" @close="showAddUserModal = false">
        <template v-slot:header>
            <h5>Add New User</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="postUser()" class="registration-form">
                <div class="form-group">
                    <label for="name">Name:</label>
                    <input class="form-control" type="text" v-model="newUserData.name" id="name" required>
                </div>
                <div class="form-group">
                    <label for="email">Email:</label>
                    <input class="form-control" type="text" v-model="newUserData.email" id="email" required>
                </div>
                <div class="form-group">
                    <label for="password">Password:</label>
                    <input class="form-control" type="password" v-model="newUserData.password" id="password" required>
                </div>
                <div class="form-group">
                    <label for="userType">User Type:</label>
                    <select class="form-control" id="userType" v-model="newUserData.userType" required>
                        <option value="admin">Administrator</option>
                        <option value="doctor">Doctor</option>
                        <option value="patient">Patient</option>
                    </select>
                </div>
                <button type="submit" class="btn btn-primary m-1">Register</button>
            </form>
            <div v-if="error" class="error">{{ error }}</div>
        </template>
    </Modal>

    <Modal v-if="showeditUserModal" @close="showeditUserModal = false">
        <template v-slot:header>
            <h5>Edit Doctor Details</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="updateUser()" class="registration-form">
                <div class="form-group">
                    <label for="name">Name:</label>
                    <input class="form-control" type="text" v-model="editUserData.name" id="name" required>
                </div>
                <div class="form-group">
                    <label for="email">Email:</label>
                    <input class="form-control" type="text" v-model="editUserData.email" id="email" required>
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
                <button type="submit" class="btn btn-primary m-1">Update</button>
            </form>
        </template>
    </Modal>
</template>



<script>
import Modal from './PopUpModal.vue';
import axios from 'axios';
import DashboardView from '@/views/DashboardView.vue';

export default {
    name: 'DoctorManagement',
    components: {
        Modal,
        DashboardView
    },
    data() {
        return {
            users: [],
            showAddUserModal: false,
            showeditUserModal: false,
            newUserData: {
                name: '',
                email: '',
                password: '',
                userType: ''
            },
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
    mounted() {
        this.fetchAllUsers();
    },
    methods: {
        fetchAllUsers() {
            fetch('http://127.0.0.1:8000/api/user', {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                },
            })
                .then(response => response.json())
                .then(data => {
                    this.users = data.UserAccounts;
                })
                .catch(err => {
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
                });
        },
        addDoctor() {
            this.showAddUserModal = true;
        },
        async postUser() {
            try {
                await axios.post('http://127.0.0.1:8000/api/user', this.newUserData, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
                this.showAddUserModal = false;
            } catch (error) {
                console.error('There was an error adding the product:', error);
            }
            this.fetchAllUsers();
        },
        editUser(user) {
            this.editUserData = { ...user };
            this.showeditUserModal = true;
        },
        async updateUser() {
            this.editUserData.updated_at = new Date().toISOString();
            await axios.put(`http://127.0.0.1:8000/api/doctor/${this.editUserData.id}`, this.editUserData, {
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                }
            });
            this.showeditUserModal = false;
            this.fetchAllUsers();
        },
        async deleteUser(doctor) {
            const index = this.doctors.findIndex(d => d.id === doctor.id);
            if (index !== -1) {
                this.doctors.splice(index, 1);
            }
            try {
                await axios.delete(`http://127.0.0.1:8000/api/doctor/${doctor.id}`, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
            } catch (error) {
                console.error('There was an error deleting the user:', error);
            }
        }
    }
};
</script>
