<template>
  <section id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autocomplete="off"
        autofocus
        v-model.trim="event"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main" v-show="count">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="allDone"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="todo in filterTodos"
          :key="todo"
          :class="{ editing: editStatus === todo, completed: todo.completed }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label @dblclick="editing(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="remove(todo)"></button>
          </div>
          <input
            class="edit"
            type="text"
            v-edit-focus="editStatus === todo"
            v-model.trim="todo.text"
            @keyup.enter="doneEdit(todo)"
            @blur="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="count">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong>
        {{ remainingCount > 1 ? "items" : "item" }} left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button
        class="clear-completed"
        @click="removeCompleted"
        v-show="count > remainingCount"
      >
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <!-- Remove the below line ↓ -->
    <p>Template by <a href="http://sindresorhus.com">Sindre Sorhus</a></p>
    <!-- Change this out with your name and url ↓ -->
    <p>Created by <a href="https://github.com/lorraineliang">lorrainejy</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</template>

<script>
import { computed, ref, onMounted, onUnmounted, watchEffect } from 'vue'
import './assets/index.css'
import useLocalStorage from './utils/useLocalStorage'

const storage = useLocalStorage()

// 1.添加待办事项
const useAdd = todos => {
  const event = ref('')
  const addTodo = () => {
    const text = event.value // ref创建的响应式对象的值存在对象仅有的value属性上
    if (text.length === 0) return
    todos.value.unshift({
      // id: todos.length,
      text,
      completed: false
    })
    event.value = ''
  }
  return {
    event,
    addTodo
  }
}

// 2. 删除待办事项
const useRemove = todos => {
  const remove = todo => {
    const index = todos.value.indexOf(todo)
    todos.value.splice(index, 1)
  }
  const removeCompleted = () => {
    console.log(todos)
    todos.value = todos.value.filter(todo => !todo.completed)
  }
  return {
    remove,
    removeCompleted
  }
}

// 3.编辑待办事项
const useEdit = remove => {
  let beforeEditText = '' // 编辑前文本内容
  const editStatus = ref(null) // 编辑状态

  const editing = todo => {
    beforeEditText = todo.text // 把编辑前文本记录下来
    editStatus.value = todo // 改变编辑状态
  }

  const doneEdit = todo => {
    if (!editStatus.value) return
    todo.text || remove(todo) // 编辑文本框但清空内容 删除当前todo
    editStatus.value = null // 完成编辑 编辑状态恢复初始化
  }

  const cancelEdit = todo => {
    todo.text = beforeEditText // 把原先记录下来的文本内容重新赋值给input框绑定的todo.text
    editStatus.value = null // 取消编辑 编辑状态恢复初始化
  }

  return {
    editStatus,
    editing,
    doneEdit,
    cancelEdit
  }
}

// 4.切换待办项完成状态等
const useCompleted = todos => {
  const allDone = computed({
    get () {
      return todos.value.every(i => i.completed)
    },
    set (value) {
      todos.value.forEach(i => {
        i.completed = value
      })
    }
  })

  const filter = {
    all: list => list, // 所有
    active: list => list.filter(todo => !todo.completed), // 未完成
    completed: list => list.filter(todo => todo.completed) // 已完成
  }
  const type = ref('all')
  const filterTodos = computed(() => filter[type.value](todos.value))
  const remainingCount = computed(() => filter.active(todos.value).length)
  const count = computed(() => todos.value.length)
  const onHashChange = () => {
    const hash = window.location.hash.replace('#/', '')
    if (filter[hash]) {
      type.value = hash
    } else {
      type.value = 'all'
      window.location.hash = ''
    }
  }
  onMounted(() => {
    window.addEventListener('hashchange', onHashChange)
    onHashChange()
  })

  onUnmounted(() => {
    window.removeEventListener('hashchange', onHashChange)
  })

  return {
    allDone,
    count,
    filterTodos,
    remainingCount
  }
}

// 5.本地存储待办事项
const useStorage = () => {
  const KEY = 'TODOKEYS'
  const todos = ref(storage.getItem(KEY) || [])
  watchEffect(() => { // 监听todos的变化
    storage.setItem(KEY, todos.value)
  })
  return todos
}

export default {
  name: 'App',
  setup () {
    const todos = useStorage()
    const { remove, removeCompleted } = useRemove(todos)
    return {
      todos,
      remove,
      removeCompleted,
      ...useEdit(remove),
      ...useAdd(todos),
      ...useCompleted(todos)
    }
  },
  directives: {
    editFocus: (el, binding) => {
      binding.value && el.focus()
    }
  }
}
</script>

<style>
</style>
