<template>
  <div>
    <!-- Header Section -->
    <div class="student-management">
      <div class="student-management__header">
        <h2>Quản lý <strong>sinh viên</strong></h2>
        <button
          class="btn-add-student"
          @click="openAddStudentModal"
          data-bs-toggle="modal"
          data-bs-target="#exampleModal"
        >
          <i class="bi bi-plus-circle"></i> Thêm mới sinh viên
        </button>
      </div>

      <!-- Student Table -->
      <table class="student-table">
        <thead>
          <tr>
            <th><input type="checkbox" /></th>
            <th>Tên sinh viên</th>
            <th>Email</th>
            <th>Địa chỉ</th>
            <th>Số điện thoại</th>
            <th>Lựa chọn</th>
          </tr>
        </thead>
        <tbody v-for="student in students" :key="student.id">
          <tr>
            <td><input type="checkbox" /></td>
            <td>{{ student.student_name }}</td>
            <td>{{ student.email }}</td>
            <td>{{ student.address }}</td>
            <td>{{ student.phone }}</td>
            <td>
              <button class="btn-edit" @click="editStudent(student)">
                <i class="bi bi-pencil"></i>
              </button>
              <button
                class="btn-delete"
                data-bs-toggle="modal"
                data-bs-target="#deleteModal"
                @click="confirmDelete(student)"
              >
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Pagination -->
      <div class="pagination-container">
        <p>Hiển thị 5/10 bản ghi</p>
        <ul class="pagination">
          <li><a href="#">Trước</a></li>
          <li><a href="#">1</a></li>
          <li><a href="#">2</a></li>
          <li class="active"><a href="#">3</a></li>
          <li><a href="#">4</a></li>
          <li><a href="#">5</a></li>
          <li><a href="#">Sau</a></li>
        </ul>
      </div>

      <div
        class="modal fade"
        id="exampleModal"
        tabindex="-1"
        aria-labelledby="exampleModalLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel">
                {{
                  isEditing ? "Sửa thông tin sinh viên" : "Thêm mới sinh viên"
                }}
              </h5>
              <button
                type="button"
                class="btn-close"
                data-bs-dismiss="modal"
                aria-label="Close"
              ></button>
            </div>
            <div class="modal-body">
              <form>
                <div class="mb-3">
                  <label for="student-name" class="col-form-label"
                    >Tên sinh viên</label
                  >
                  <input
                    v-model="newStudent.student_name"
                    type="text"
                    class="form-control"
                    id="student-name"
                    required
                  />
                  <div v-if="errors.student_name" class="text-danger">
                    {{ errors.student_name }}
                  </div>
                </div>
                <div class="mb-3">
                  <label for="email" class="col-form-label">Email</label>
                  <input
                    v-model="newStudent.email"
                    type="email"
                    class="form-control"
                    id="email"
                    required
                  />
                  <div v-if="errors.email" class="text-danger">
                    {{ errors.email }}
                  </div>
                </div>
                <div class="mb-3">
                  <label for="address" class="col-form-label">Địa chỉ</label>
                  <textarea
                    v-model="newStudent.address"
                    class="form-control"
                    id="address"
                    required
                  ></textarea>
                </div>
                <div class="mb-3">
                  <label for="phone" class="col-form-label"
                    >Số điện thoại</label
                  >
                  <input
                    v-model="newStudent.phone"
                    type="text"
                    class="form-control"
                    id="phone"
                    required
                  />
                  <div v-if="errors.phone" class="text-danger">
                    {{ errors.phone }}
                  </div>
                </div>
              </form>
            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn btn-secondary"
                data-bs-dismiss="modal"
              >
                Cancel
              </button>
              <button type="button" class="btn btn-primary" @click="addStudent">
                {{ isEditing ? "Update" : "Add" }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <div>
        <!-- Modal Xóa Sinh Viên -->
        <div
          class="modal fade"
          id="deleteModal"
          tabindex="-1"
          aria-labelledby="deleteModalLabel"
          aria-hidden="true"
        >
          <div class="modal-dialog">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="deleteModalLabel">Xác nhận xóa</h5>
                <button
                  type="button"
                  class="btn-close"
                  data-bs-dismiss="modal"
                  aria-label="Close"
                ></button>
              </div>
              <div class="modal-body">
                Bạn có chắc chắn muốn xóa sinh viên "{{
                  selectedStudent.student_name
                }}"?
              </div>
              <div class="modal-footer">
                <button
                  type="button"
                  class="btn btn-secondary"
                  data-bs-dismiss="modal"
                >
                  Cancel
                </button>
                <button
                  type="button"
                  class="btn btn-danger"
                  @click="deleteStudent"
                >
                  Xóa
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { onMounted, reactive, ref } from "vue";

// Danh sách sinh viên
const students = ref([]);
// Quản lý chế độ thêm/sửa sinh viên
const isEditing = ref(false);
// Biến lưu sinh viên đang được chọn để xóa
const selectedStudent = reactive({});
// ID của sinh viên đang được chọn để sửa
const selectedStudentId = ref(null);

// Lấy dữ liệu danh sách sinh viên
const fetchData = async () => {
  try {
    const response = await axios.get("http://localhost:8080/students");
    students.value = response.data;
  } catch (error) {
    console.error("Error fetching students:", error);
  }
};

onMounted(() => {
  fetchData();
});

// Đối tượng sinh viên mới
const newStudent = reactive({
  student_name: "",
  email: "",
  address: "",
  phone: "",
  status: true,
  created_at: new Date().toISOString(),
});

// Đối tượng lưu lỗi khi nhập form
const errors = reactive({
  student_name: "",
  email: "",
  phone: "",
});

// Hàm kiểm tra form
const validateForm = () => {
  let isValid = true;

  // Kiểm tra tên sinh viên
  if (!newStudent.student_name) {
    errors.student_name = "Tên sinh viên không được để trống";
    isValid = false;
  } else {
    errors.student_name = "";
  }

  // Kiểm tra email
  const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!newStudent.email) {
    errors.email = "Email không được để trống";
    isValid = false;
  } else if (!emailPattern.test(newStudent.email)) {
    errors.email = "Email không đúng định dạng";
    isValid = false;
  } else {
    errors.email = "";
  }

  // Kiểm tra số điện thoại (chỉ cho phép nhập số)
  const phonePattern = /^[0-9]*$/;
  if (!newStudent.phone) {
    errors.phone = ""; // Cho phép để trống số điện thoại
  } else if (!phonePattern.test(newStudent.phone)) {
    errors.phone = "Số điện thoại chỉ được phép chứa chữ số";
    isValid = false;
  } else {
    errors.phone = "";
  }

  return isValid;
};

// Hàm mở modal thêm sinh viên
const openAddStudentModal = () => {
  isEditing.value = false;
  resetForm();
  resetErrors();
};

// Hàm thêm sinh viên
const addStudent = async () => {
  if (!validateForm()) {
    return;
  }

  try {
    if (isEditing.value && selectedStudentId.value) {
      // Sửa sinh viên
      await axios.put(
        `http://localhost:8080/students/${selectedStudentId.value}`,
        newStudent
      );
    } else {
      // Thêm sinh viên mới
      await axios.post("http://localhost:8080/students", newStudent);
    }

    fetchData();
    resetForm();
    hideModal("exampleModal"); // Ẩn modal sau khi thêm
  } catch (error) {
    console.error("Error saving student:", error);
  }
};

// Hàm reset form
const resetForm = () => {
  newStudent.student_name = "";
  newStudent.email = "";
  newStudent.address = "";
  newStudent.phone = "";
  isEditing.value = false;
  selectedStudentId.value = null;
  resetErrors();
};

// Hàm sửa sinh viên
const editStudent = (student) => {
  isEditing.value = true;
  selectedStudentId.value = student.id;
  newStudent.student_name = student.student_name;
  newStudent.email = student.email;
  newStudent.address = student.address;
  newStudent.phone = student.phone;
  resetErrors();
  showModal("exampleModal"); // Hiển thị modal
};

// Hàm reset lỗi
const resetErrors = () => {
  errors.student_name = "";
  errors.email = "";
  errors.phone = "";
};

// Hàm hiển thị modal
const showModal = (modalId) => {
  const modalElement = document.getElementById(modalId);
  const modalInstance = new bootstrap.Modal(modalElement);
  modalInstance.show();
};

// Hàm ẩn modal
const hideModal = (modalId) => {
  const modalElement = document.getElementById(modalId);
  const modalInstance = bootstrap.Modal.getInstance(modalElement);
  if (modalInstance) modalInstance.hide();
};

// Khi nhấn nút xóa, mở modal và lưu thông tin sinh viên
const confirmDelete = (student) => {
  selectedStudent.id = student.id;
  selectedStudent.student_name = student.student_name;
};

// Thực hiện xóa sinh viên
const deleteStudent = async () => {
  try {
    await axios.delete(`http://localhost:8080/students/${selectedStudent.id}`);
    fetchData();
    // Đóng modal xóa
    const modalElement = document.getElementById("deleteModal");
    const modalInstance = bootstrap.Modal.getInstance(modalElement);
    modalInstance.hide();
  } catch (error) {
    console.error(error);
  }
};
</script>

<style scoped>
/* Container styling */
.student-management {
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 20px;
  background-color: #f8f9fa;
}

/* Header section styling */
.student-management__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.student-management__header h2 {
  font-size: 24px;
  color: #2c3e50;
}

.btn-add-student {
  background-color: #28a745;
  color: white;
  padding: 8px 16px;
  border-radius: 5px;
  font-size: 16px;
  display: flex;
  align-items: center;
}

.btn-add-student i {
  margin-right: 8px;
  font-size: 18px;
}

/* Table styling */
.student-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 10px;
}

.student-table th,
.student-table td {
  padding: 12px;
  text-align: left;
  border: 1px solid #ddd;
}

.student-table th {
  background-color: #e9ecef;
  font-weight: bold;
  color: #333;
}

.student-table td {
  background-color: #fff;
}

/* Button styling */
.btn-edit {
  background-color: #ffc107;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 3px;
  margin-right: 5px;
}

.btn-delete {
  background-color: #dc3545;
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 3px;
}

/* Pagination styling */
.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  font-size: 14px;
}

.pagination {
  list-style: none;
  display: flex;
  gap: 5px;
  padding-left: 0;
}

.pagination li a {
  color: #007bff;
  text-decoration: none;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 3px;
}

.pagination .active a {
  background-color: #007bff;
  color: white;
  border-color: #007bff;
}

.text-danger {
  color: red;
  font-size: 16px;
}
</style>
