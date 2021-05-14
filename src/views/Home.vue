<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <!--Bindeamos en el prop del componente Tasks la
        data que hemos cargado en el metodo created-->
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    }
  },
  methods: {
     /*Usando el operador Spread across añadimos el nuevo
    task, recibido en el emit del componente AddTask, en la
    lista de tasks
    */
    async addTask(task) {
      const res = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(task),
      })
      const data = await res.json()
      this.tasks = [...this.tasks, data]
    },
    /*Usando el metodo filter podemos iterar en la lista de
    en cada task para eliminar aquel que corresponda a la id
    que recibe nuestro metodo
    */
    async deleteTask(id) {
      if (confirm('Are you sure?')) {
        const res = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        })
        res.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert('Error deleting task')
      }
    },
    /*Usando el metodo map podemos recorrer y retornar un arreglo
    con el task modificado. Con la notacion ...task aseguramos que solo
    modificamos un parametro en particular de nuestro task (osea el id)
    y esto solo aplica para el task al cual le dimos click
    */
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id)
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder }
      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updTask),
      })
      const data = await res.json()
      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      )
    },
    async fetchTasks() {
      const res = await fetch('api/tasks')
      const data = await res.json()
      return data
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`)
      const data = await res.json()
      return data
    },

  },
    /*Se especifica que en el ciclo de vida del componente,
    en la etapa "created", se llenara el objeto que tenemos
    en data() con unas Tasks de prueba. En un caso normal, 
    donde se tenga una API, esta carga de informacion se 
    manejaria por medio de una peticion HTTP*/
    async created() {
      this.tasks = await this.fetchTasks()
    },
};
</script>