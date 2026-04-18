<template>
  <div class="container">
    <h1>待办事项管理系统</h1>

    <!-- 登录/注册区域 -->
    <div v-if="!isLoggedIn" class="auth">
      <h2>{{ isLoginMode ? '登录' : '注册' }}</h2>
      <input type="text" v-model="username" placeholder="用户名" />
      <input type="password" v-model="password" placeholder="密码" />
      <button @click="submitAuth">{{ isLoginMode ? '登录' : '注册' }}</button>
      <a @click="isLoginMode = !isLoginMode">
        {{ isLoginMode ? '没有账号？去注册' : '已有账号？去登录' }}
      </a>
    </div>

    <!-- 待办列表区域（登录后显示） -->
    <div v-else>
      <div class="add-todo">
        <input type="text" v-model="newTodoTitle" placeholder="添加新待办" @keyup.enter="addTodo" />
        <button @click="addTodo">添加</button>
      </div>

      <ul class="todo-list">
        <li v-for="todo in todos" :key="todo.id">
          <input type="checkbox" v-model="todo.completed" @change="toggleTodo(todo)" />
          <span :class="{ completed: todo.completed }">{{ todo.title }}</span>
          <button @click="deleteTodo(todo.id)">删除</button>
        </li>
      </ul>

      <button @click="logout">退出登录</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      isLoginMode: true,
      isLoggedIn: false,
      username: '',
      password: '',
      userId: null,
      todos: [],
      newTodoTitle: ''
    }
  },
  methods: {
    async submitAuth() {
      const url = this.isLoginMode
          ? 'http://10.126.232.229:8080/api/user/login'
          : 'http://10.126.232.229:8080/api/user/register'

      try {
        const response = await axios.post(url, null, {
          params: {
            username: this.username,
            password: this.password
          }
        })

        const result = response.data
        if (this.isLoginMode) {
          if (result.includes('登录成功')) {
            const match = result.match(/用户ID：(\d+)/)
            if (match) {
              this.userId = parseInt(match[1])
              this.isLoggedIn = true
              this.loadTodos()
              alert('登录成功')
            }
          } else {
            alert(result)
          }
        } else {
          alert(result)
          this.isLoginMode = true
        }
      } catch (error) {
        alert('请求失败：' + error.message)
      }
    },

    async loadTodos() {
      try {
        const response = await axios.get('http://10.126.232.229:8080/api/todo/list', {
          params: { userId: this.userId }
        })
        this.todos = response.data
      } catch (error) {
        alert('加载待办失败')
      }
    },

    async addTodo() {
      if (!this.newTodoTitle.trim()) return
      try {
        const response = await axios.post('http://10.126.232.229:8080/api/todo/add', null, {
          params: {
            userId: this.userId,
            title: this.newTodoTitle
          }
        })
        this.todos.push(response.data)
        this.newTodoTitle = ''
      } catch (error) {
        alert('添加失败')
      }
    },

    async toggleTodo(todo) {
      try {
        await axios.put('http://10.126.232.229:8080/api/todo/toggle', null, {
          params: { id: todo.id }
        })
      } catch (error) {
        alert('更新失败')
        todo.completed = !todo.completed
      }
    },

    async deleteTodo(id) {
      try {
        await axios.delete('http://10.126.232.229:8080/api/todo/delete', {
          params: { id: id }
        })
        this.todos = this.todos.filter(t => t.id !== id)
      } catch (error) {
        alert('删除失败')
      }
    },

    logout() {
      this.isLoggedIn = false
      this.userId = null
      this.username = ''
      this.password = ''
      this.todos = []
    }
  }
}
</script>