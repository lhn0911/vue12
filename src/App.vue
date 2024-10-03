<template>
  <div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2 class="mb-0">Quản lý mượn trả sách</h2>
      <div>
        <select
          v-model="statusFilter"
          class="form-select me-2"
          style="width: auto"
        >
          <option value="all">Lọc theo trạng thái</option>
          <option value="returned">Đã trả</option>
          <option value="not-returned">Chưa trả</option>
        </select>
        <button class="btn btn-primary" @click="openForm()">
          Thêm thông tin
        </button>
      </div>
    </div>

    <div class="table-responsive">
      <table class="table table-bordered table-hover">
        <thead class="table-light">
          <tr>
            <th>STT</th>
            <th>Tên sách</th>
            <th>Sinh viên mượn</th>
            <th>Ngày mượn</th>
            <th>Ngày trả</th>
            <th>Trạng thái</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(book, index) in filteredBooks" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ book.title }}</td>
            <td>{{ book.student }}</td>
            <td>{{ formatDate(book.borrowDate) }}</td>
            <td>{{ formatDate(book.returnDate) }}</td>
            <td>
              <button
                :class="[
                  'btn badge',
                  book.returned ? 'bg-success' : 'bg-danger',
                ]"
                @click="toggleBookStatus(index)"
              >
                {{ book.returned ? "Đã trả" : "Chưa trả" }}
              </button>
            </td>
            <td>
              <button
                class="btn btn-warning btn-sm me-1"
                @click="openForm(index)"
              >
                Sửa
              </button>
              <button
                class="btn btn-danger btn-sm"
                @click="confirmDelete(index)"
              >
                Xóa
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Form thêm/sửa sách -->
    <ModalForm
      v-if="showForm"
      :is-editing="isEditing"
      :current-book="currentBook"
      @close="closeForm"
      @submit="submitForm"
    />

    <!-- Modal xác nhận xóa -->
    <ModalConfirm
      v-if="showDeleteModal"
      @confirm="deleteBook"
      @cancel="showDeleteModal = false"
    />
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import ModalForm from "./ModalForm.vue"; // Đặt form vào một component riêng
import ModalConfirm from "./ModalConfirm.vue"; // Tương tự với modal xác nhận

const books = ref(JSON.parse(localStorage.getItem("books")) || []);
const statusFilter = ref("all");
const showForm = ref(false);
const showDeleteModal = ref(false);
const isEditing = ref(false);
const currentBook = ref({
  title: "",
  student: "",
  borrowDate: "",
  returnDate: "",
  returned: false,
});
let bookToEdit = ref(null);
let bookToDelete = ref(null);

const formatDate = (date) => new Date(date).toLocaleDateString("vi-VN");

// Lọc sách theo trạng thái
const filteredBooks = computed(() =>
  statusFilter.value === "all"
    ? books.value
    : books.value.filter(
        (book) => (statusFilter.value === "returned") === book.returned
      )
);

// Hiển thị form thêm/sửa
const openForm = (index = null) => {
  isEditing.value = index !== null;
  currentBook.value =
    index !== null
      ? { ...books.value[index] }
      : {
          title: "",
          student: "",
          borrowDate: "",
          returnDate: "",
          returned: false,
        };
  bookToEdit.value = index;
  showForm.value = true;
};

// Đóng form
const closeForm = () => (showForm.value = false);

// Xử lý submit form (thêm hoặc sửa)
const submitForm = () => {
  if (currentBook.value.borrowDate > currentBook.value.returnDate)
    return alert("Ngày trả không được nhỏ hơn ngày mượn.");

  if (isEditing.value) books.value[bookToEdit.value] = { ...currentBook.value };
  else books.value.push({ ...currentBook.value });

  saveBooks();
  closeForm();
};

// Hiển thị modal xác nhận xóa
const confirmDelete = (index) => {
  bookToDelete.value = index;
  showDeleteModal.value = true;
};

// Xóa sách
const deleteBook = () => {
  books.value.splice(bookToDelete.value, 1);
  saveBooks();
  showDeleteModal.value = false;
};

// Thay đổi trạng thái sách
const toggleBookStatus = (index) => {
  books.value[index].returned = !books.value[index].returned;
  saveBooks();
};

// Lưu sách vào localStorage
const saveBooks = () =>
  localStorage.setItem("books", JSON.stringify(books.value));
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
