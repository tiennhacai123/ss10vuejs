<template>
  <div class="container">
    <header class="header">
      <h3>Nhân viên</h3>
      <button class="btn-add" @click="showAddModal">Thêm mới nhân viên</button>
    </header>

    <!-- Input tìm kiếm theo email -->
    <div class="search">
      <input
        type="text"
        placeholder="Tìm kiếm theo email..."
        v-model="searchQuery"
        @input="filterEmployees"
      />
    </div>

    <!-- Modal thêm mới nhân viên -->
    <div class="modal" v-if="showModal">
      <form class="form" @submit.prevent="submitEmployee">
        <div class="modal-header">
          <h4>{{ currentEmployeeId ? 'Chỉnh sửa nhân viên' : 'Thêm mới nhân viên' }}</h4>
          <i class="fa-solid fa-xmark close-icon" @click="closeModal"></i>
        </div>

        <div class="form-group">
          <label for="userName">Họ và tên</label>
          <input id="userName" type="text" v-model="employeeForm.name" required />
          <span v-if="errors.name" class="error">{{ errors.name }}</span>
        </div>

        <div class="form-group">
          <label for="dateOfBirth">Ngày sinh</label>
          <input id="dateOfBirth" type="date" v-model="employeeForm.birthDate" required />
          <span v-if="errors.birthDate" class="error">{{ errors.birthDate }}</span>
        </div>

        <div class="form-group">
          <label for="email">Email</label>
          <input id="email" type="email" v-model="employeeForm.email" required />
          <span v-if="errors.email" class="error">{{ errors.email }}</span>
        </div>

        <div class="form-group">
          <label for="address">Địa chỉ</label>
          <textarea id="address" v-model="employeeForm.address" rows="3"></textarea>
        </div>

        <div class="form-group">
          <button class="btn-submit">{{ currentEmployeeId ? 'Cập nhật' : 'Thêm mới' }}</button>
        </div>
      </form>
    </div>

    <div v-if="filteredEmployees.length">
      <h4>Danh sách nhân viên</h4>
      <table class="employee-table">
        <thead>
          <tr>
            <th>Họ và tên</th>
            <th>Email</th>
            <th>Ngày sinh</th>
            <th>Địa chỉ</th>
            <th>Hành động</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="employee in filteredEmployees" :key="employee.id">
            <td>{{ employee.name }}</td>
            <td>{{ employee.email }}</td>
            <td>{{ employee.birthDate }}</td>
            <td>{{ employee.address }}</td>
            <td>
              <button class="btn-edit" @click="editEmployee(employee.id)">Chỉnh sửa</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const employees = ref([
      { id: 1, name: 'Nguyễn Văn A', birthDate: '1990-02-28', email: 'nvana@gmail.com', address: 'Ba Đình, Hà Nội', status: 'status-active' },
      { id: 2, name: 'Trần Thị B', birthDate: '1985-07-15', email: 'ttb@gmail.com', address: 'Cầu Giấy, Hà Nội', status: 'status-stop' },
      { id: 3, name: 'Lê Văn C', birthDate: '2000-10-03', email: 'lvc@gmail.com', address: 'Hai Bà Trưng, Hà Nội', status: 'status-stop' },
      { id: 4, name: 'Phạm Thị D', birthDate: '1995-05-20', email: 'ptd@gmail.com', address: 'Hoàn Kiếm, Hà Nội', status: 'status-active' },
      { id: 5, name: 'Ngô Văn E', birthDate: '1988-11-12', email: 'nve@gmail.com', address: 'Cầu Giấy, Hà Nội', status: 'status-active' }
    ]);

    const showModal = ref(false);
    const employeeForm = ref({ name: '', birthDate: '', email: '', address: '' });
    const currentEmployeeId = ref(null);
    const errors = ref({ name: '', birthDate: '', email: '' });
    const searchQuery = ref('');
    const filteredEmployees = ref(employees.value);

    const showAddModal = () => {
      employeeForm.value = { name: '', birthDate: '', email: '', address: '' };
      errors.value = { name: '', birthDate: '', email: '' };
      showModal.value = true;
      currentEmployeeId.value = null;
    };

    const validateForm = () => {
      let valid = true;
      errors.value = { name: '', birthDate: '', email: '' };

      if (!employeeForm.value.name) {
        errors.value.name = 'Họ và tên không được để trống.';
        valid = false;
      }
      if (!employeeForm.value.email) {
        errors.value.email = 'Email không được để trống.';
        valid = false;
      } else if (!/\S+@\S+\.\S+/.test(employeeForm.value.email)) {
        errors.value.email = 'Email không đúng định dạng.';
        valid = false;
      }
      if (!employeeForm.value.birthDate) {
        errors.value.birthDate = 'Ngày sinh không được để trống.';
        valid = false;
      } else if (new Date(employeeForm.value.birthDate) > new Date()) {
        errors.value.birthDate = 'Ngày sinh không được lớn hơn ngày hiện tại.';
        valid = false;
      }

      return valid;
    };

    const submitEmployee = () => {
      if (validateForm()) {
        if (currentEmployeeId.value) {
          const index = employees.value.findIndex(emp => emp.id === currentEmployeeId.value);
          if (index !== -1) {
            employees.value[index] = { ...employeeForm.value, id: currentEmployeeId.value };
          }
        } else {
          const newId = employees.value.length ? Math.max(...employees.value.map(emp => emp.id)) + 1 : 1;
          employees.value.push({ ...employeeForm.value, id: newId });
        }
        closeModal();
        filterEmployees();
      }
    };

    const editEmployee = (id) => {
      const employee = employees.value.find(emp => emp.id === id);
      if (employee) {
        employeeForm.value = { ...employee };
        currentEmployeeId.value = id;
        showModal.value = true;
      }
    };

    const closeModal = () => {
      showModal.value = false;
      employeeForm.value = { name: '', birthDate: '', email: '', address: '' };
      errors.value = { name: '', birthDate: '', email: '' };
    };

    const filterEmployees = () => {
      filteredEmployees.value = employees.value.filter(employee =>
        employee.email.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    };

    return {
      employees,
      showModal,
      employeeForm,
      errors,
      searchQuery,
      filteredEmployees,
      showAddModal,
      submitEmployee,
      editEmployee,
      closeModal,
      filterEmployees,
    };
  },
};
</script>

<style scoped>
.container {
  width: 80%;
  margin: 0 auto;
  padding-top: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.btn-add {
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-add:hover {
  background-color: #0056b3;
}

.search {
  margin-bottom: 20px;
}

.search input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.form {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.close-icon {
  cursor: pointer;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-group textarea {
  resize: none;
}

.btn-submit {
  padding: 10px 15px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-submit:hover {
  background-color: #218838;
}

.error {
  color: red;
  font-size: 12px;
}

.employee-table {
  width: 100%;
  border-collapse: collapse;
}

.employee-table th,
.employee-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.employee-table th {
  background-color: #f2f2f2;
}
</style>
