<template>
    <DashboardView/>
    <div class="hello">
        <div class="d-flex justify-content-center align-items-center mb-3">
            <h1 class="text-center">Doctor Management
            <button class="btn btn-info btn-sm ml-3" @click="addDoctor()">Add a Doctor</button>
            </h1>
        </div>
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-body text-center">
                        <h5 class="card-title">-- Doctors List --</h5>
                        <div v-for="doctor in doctors" :key="doctor.id" class="card mb-2">
                            <div class="card-body">
                                <h6 class="card-subtitle mb-2 text-muted">ID: {{ doctor.id }}</h6>
                                <p class="card-text">Name: {{ doctor.name }}</p>
                                <p class="card-text">Email: {{ doctor.email }}</p>
                                <button class="btn btn-warning btn-sm m-1" @click="editDoctor(doctor)">Edit</button>
                                <button class="btn btn-danger btn-sm m-1" @click="deleteDoctor(doctor)">Delete</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <Modal v-if="showAddDoctorModal" @close="showAddDoctorModal = false">
        <template v-slot:header>
            <h5>Add New Doctor</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="postDoctor">
                <div class="form-group">
                    <label for="newDoctorName">Name</label>
                    <input type="text" class="form-control" id="newDoctorName" v-model="newDoctorData.name">
                </div>
                <div class="form-group">
                    <label for="newDoctorEmail">Email</label>
                    <input type="text" class="form-control" id="newDoctorEmail" v-model="newDoctorData.email">
                </div>
                <div class="form-group">
                    <label for="newDoctorPassword">Password</label>
                    <input type="password" class="form-control" id="newDoctorPassword" v-model="newDoctorData.password">
                </div>
                <button type="submit" class="btn btn-primary">Add</button>
            </form>
        </template>
    </Modal>

    <Modal v-if="showEditDoctorModal" @close="showEditDoctorModal = false">
        <template v-slot:header>
            <h5>Edit Doctor Details</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="updateDoctor()">
                <div class="form-group">
                    <label for="editDoctorName">Name</label>
                    <input type="text" class="form-control" id="editDoctorName" v-model="editDoctorData.name">
                </div>
                <div class="form-group">
                    <label for="editDoctorEmail">Email</label>
                    <input type="text" class="form-control" id="editDoctorEmail" v-model="editDoctorData.email">
                </div>
                <div class="form-group">
                    <label for="editDoctorPassword">Password</label>
                    <input type="password" class="form-control" id="editDoctorPassword" v-model="editDoctorData.password">
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
            doctors: [],
            showAddDoctorModal: false,
            showEditDoctorModal: false,
            newDoctorData: {
                name: '',
                email: '',
                password: '',
                userType: 'doctor'
            },
            editDoctorData: {
                id: '',
                name: '',
                email: '',
                password: '',
                userType: 'doctor',
                updated_at: ''
            }
        };
    },
    mounted() {
        this.fetchDoctors();
    },
    methods: {
        fetchDoctors() {
            fetch('http://127.0.0.1:8000/api/doctor', {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                },
            })
                .then(response => response.json())
                .then(data => {
                    this.doctors = data.DoctorAccounts;
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
            this.showAddDoctorModal = true;
        },
        async postDoctor() {
            try {
                await axios.post('http://127.0.0.1:8000/api/doctor', this.newDoctorData, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
                this.showAddDoctorModal = false;
            } catch (error) {
                console.error('There was an error adding the product:', error);
            }
            this.fetchDoctors();
        },
        editDoctor(doctor) {
            this.editDoctorData = { ...doctor };
            this.showEditDoctorModal = true;
        },
        async updateDoctor() {
            this.editDoctorData.updated_at = new Date().toISOString();
            await axios.put(`http://127.0.0.1:8000/api/doctor/${this.editDoctorData.id}`, this.editDoctorData, {
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                }
            });
            this.showEditDoctorModal = false;
            this.fetchDoctors();
        },
        async deleteDoctor(doctor) {
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
                console.error('There was an error deleting the product:', error);
            }
        }
    }
};
</script>

<style scoped>
.hello {
    padding: 20px;
}

.card {
    border: 1px solid #dee2e6;
    border-radius: 10px;
    margin-bottom: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.card-title {
    font-size: 1.8rem;
    margin-bottom: 20px;
    color: #343a40;
}

.card-subtitle {
    color: #6c757d;
    margin-bottom: 5px;
}

.card-body {
    padding: 20px;
}

.btn {
    cursor: pointer;
}

.btn-primary {
    background-color: #007bff;
    border-color: #007bff;
}

.btn-primary:hover {
    background-color: #0056b3;
    border-color: #0056b3;
}

.btn-info {
    background-color: #17a2b8;
    border-color: #17a2b8;
}

.btn-info:hover {
    background-color: #117a8b;
    border-color: #117a8b;
}

.btn-warning {
    background-color: #ffc107;
    border-color: #ffc107;
}

.btn-warning:hover {
    background-color: #d39e00;
    border-color: #d39e00;
}

.btn-danger {
    background-color: #dc3545;
    border-color: #dc3545;
}

.btn-danger:hover {
    background-color: #bd2130;
    border-color: #bd2130;
}
</style>
