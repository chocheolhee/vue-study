<template>
  <div>
    <div class="d-flex justify-content-between mb-3">
      <h2>Todo List</h2>
      <button
          class="btn btn-primary"
          @click="moveToCreatePage"
      >
        Create Todo
      </button>
    </div>

    <input
        class="form-control"
        type="text"
        v-model="searchText"
        placeholder="Search"
        @keyup.enter="searchTodo"
    />

    <hr/>

    <div v-if="!todos.length">
      There is nothing to display
    </div>

    <TodoList :todos="todos" @toggle-todo="toggleTodo" @delete-todo="deleteTodo"/>

    <hr/>
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-if="currentPage !== 1" class="page-item">
          <a class="page-link" @click="getTodos(currentPage - 1)" style="cursor: pointer">
            Previous
          </a>
        </li>
        <li
            v-for="page in numberOfPages"
            :key="page"
            class="page-item"
            :class="currentPage === page ? 'active' : ''"
        >
          <a class="page-link" @click="getTodos(page)">{{ page }}</a>
        </li>
        <li v-if="numberOfPages !== currentPage" class="page-item">
          <a class="page-link" @click="getTodos(currentPage + 1)" style="cursor: pointer">
            Next
          </a>
        </li>
      </ul>
    </nav>
  </div>
  <Toast
      v-if="showToast"
      :message="toastMessage"
      :type="toastAlertType"
  />
</template>

<script>
import {ref, computed, watch} from "vue";
import TodoList from "@/components/TodoList.vue";
import axios from "axios";
import Toast from "@/components/Toast.vue";
import {useToast} from "@/composables/toast";
import {useRouter} from "vue-router";

export default {
  components: {
    Toast,
    TodoList
  },
  setup() {
    const todos = ref([]);
    const error = ref('');
    const numberOfTodos = ref(0);
    const currentPage = ref(1);
    const limit = 5;
    const searchText = ref('');
    const router = useRouter();

    const numberOfPages = computed(() => {
      return Math.ceil(numberOfTodos.value / limit);
    });

    const {
      triggerToast,
      toastMessage,
      toastAlertType,
      showToast
    } = useToast();

    const getTodos = async (page = currentPage.value) => {
      currentPage.value = page;
      try {
        const res = await axios.get(`http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${limit}`);
        numberOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        error.value = 'SomeThing went wrong';
        triggerToast('Something went wrong', 'danger');
      }
    };

    getTodos();

    const addTodo = async (todo) => {
      error.value = '';

      try {
        await axios.post('http://localhost:3000/todos/', {
          subject: todo.subject,
          completed: todo.completed
        });
        await getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = 'SomeThing went wrong';
        triggerToast('Something went wrong', 'danger');
      }
    };

    const deleteTodo = async (id) => {
      error.value = '';

      try {
        await axios.delete('http://localhost:3000/todos/' + id);
        await getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong';
        triggerToast('Something went wrong', 'danger');
      }
    };

    const toggleTodo = async (index, checked) => {
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('http://localhost:3000/todos/' + id, {
          completed: checked
        });
        todos.value[index].completed = checked
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong';
        triggerToast('Something went wrong', 'danger');
      }
    };

    const moveToCreatePage = () => {
      router.push({
        name: 'TodoCreate',

      })
    }

    let timeout = null;
    const searchTodo = () => {
      clearTimeout(timeout);
      getTodos(1);
    };

    watch(searchText, () => {
      clearTimeout(timeout);

      timeout = setTimeout(() => {
        getTodos(1);
      }, 2000)
    });

    return {
      todos,
      addTodo,
      deleteTodo,
      toggleTodo,
      searchText,
      error,
      numberOfPages,
      currentPage,
      getTodos,
      searchTodo,
      showToast,
      toastMessage,
      toastAlertType,
      triggerToast,
      moveToCreatePage
    };
  }
}
</script>

<style>

</style>