<template>
  <div
    class="modal fade"
    id="studentModal"
    tabindex="-1"
    aria-labelledby="studentModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="studentModalLabel">
            {{ isEditing ? "Sửa thông tin sinh viên" : "Thêm mới sinh viên" }}
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
              <label for="phone" class="col-form-label">Số điện thoại</label>
              <input
                v-model="newStudent.phone"
                type="text"
                class="form-control"
                id="phone"
                @input="validatePhone"
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
            @click="resetForm"
          >
            Cancel
          </button>
          <button
            type="button"
            class="btn btn-primary"
            @click="addOrUpdateStudent"
          >
            {{ isEditing ? "Update" : "Add" }}
          </button>
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
</script>

<style></style>
