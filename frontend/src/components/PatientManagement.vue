<template>
    <DashboardView />
    <div class="hello">
        <div class="d-flex justify-content-center align-items-center mb-3">
            <h1 class="text-center">Patient Management
                <button class="btn btn-info btn-sm ml-3" @click="addPatient()">Add a Patient</button>
            </h1>
        </div>
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-body text-center">
                        <h5 class="card-title">-- Patients List --</h5>
                        <div v-for="patient in patients" :key="patient.id" class="card mb-2">
                            <div class="card-body">
                                <h6 class="card-subtitle mb-2 text-muted">ID: {{ patient.id }}</h6>
                                <p class="card-text">Name: {{ patient.name }}</p>
                                <p class="card-text">Email: {{ patient.email }}</p>
                                <button v-if="isAdmin || isDoctor" class="btn btn-warning btn-sm m-1"
                                    @click="editPatient(patient)">Edit</button>
                                <button v-if="isAdmin" class="btn btn-danger btn-sm m-1"
                                    @click="deletePatient(patient)">Delete</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <Modal v-if="showAddPatientModal" @close="showAddPatientModal = false" class="custom-modal">
        <template v-slot:header>
            <h5 class="modal-title">Add New Patient</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="postPatient" class="modal-form">
                <div class="form-group">
                    <label for="newPatientName">Name</label>
                    <input type="text" class="form-control" id="newPatientName" v-model="newPatientData.name">
                </div>
                <div class="form-group">
                    <label for="newPatientEmail">Email</label>
                    <input type="text" class="form-control" id="newPatientEmail" v-model="newPatientData.email">
                </div>
                <div class="form-group">
                    <label for="newPatientPassword">Password</label>
                    <input type="password" class="form-control" id="newPatientPassword"
                        v-model="newPatientData.password">
                </div>
                <button type="submit" class="btn btn-primary">Add</button>
            </form>
        </template>
    </Modal>

    <Modal v-if="showEditPatientModal" @close="showEditPatientModal = false">
        <template v-slot:header>
            <h5>Edit Patient Details</h5>
        </template>
        <template v-slot:body>
            <form @submit.prevent="updatePatient()">
                <div class="form-group">
                    <label for="editPatientName">Name</label>
                    <input type="text" class="form-control" id="editPatientName" v-model="editPatientData.name">
                </div>
                <div class="form-group">
                    <label for="editPatientEmail">Email</label>
                    <input type="text" class="form-control" id="editPatientEmail" v-model="editPatientData.email">
                </div>
                <div class="form-group">
                    <label for="editPatientPassword">Password</label>
                    <input type="password" class="form-control" id="editPatientPassword"
                        v-model="editPatientData.password">
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
    name: 'PatientManagement',
    components: {
        Modal,
        DashboardView
    },
    data() {
        return {
            patients: [],
            user: '',
            showAddPatientModal: false,
            showEditPatientModal: false,
            newPatientData: {
                name: '',
                email: '',
                password: '',
                userType: 'patient'
            },
            editPatientData: {
                id: '',
                name: '',
                email: '',
                password: '',
                userType: 'patient',
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
        },
    },
    mounted() {
        this.fetchPatients();
        this.loadUserFromLocalStorage();
    },
    methods: {
        fetchPatients() {
            fetch('http://127.0.0.1:8000/api/patient', {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                },
            })
                .then(response => response.json())
                .then(data => {
                    this.patients = data.PatientAccounts;
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
        addPatient() {
            this.showAddPatientModal = true;
        },
        async postPatient() {
            try {
                await axios.post('http://127.0.0.1:8000/api/patient', this.newPatientData, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
                this.showAddPatientModal = false;
            } catch (error) {
                console.error('There was an error adding the patient:', error);
            }
            this.fetchPatients();
        },
        editPatient(patient) {
            this.editPatientData = { ...patient };
            this.showEditPatientModal = true;
        },
        async updatePatient() {
            this.editPatientData.updated_at = new Date().toISOString();
            await axios.put(`http://127.0.0.1:8000/api/patient/${this.editPatientData.id}`, this.editPatientData, {
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer ' + localStorage.getItem('token')
                }
            });
            this.showEditPatientModal = false;
            this.fetchPatients();
        },
        async deletePatient(patient) {
            const index = this.patients.findIndex(p => p.id === patient.id);
            if (index !== -1) {
                this.patients.splice(index, 1);
            }
            try {
                await axios.delete(`http://127.0.0.1:8000/api/patient/${patient.id}`, {
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                });
            } catch (error) {
                console.error('There was an error deleting the patient:', error);
            }
        },
        loadUserFromLocalStorage() {
            const user = localStorage.getItem('user');
            if (user) {
                this.user = JSON.parse(user);
            }
        },
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
