<script setup>
import VueDatePicker from '@vuepic/vue-datepicker';
import TodoContainer from '@/components/TodoContainer.vue';
import { ref, computed, nextTick, onMounted, onBeforeMount } from 'vue';
const addingToDo = ref(false);
const newTodoName = ref('')
const newTodoInput = ref(null);
const newFullDateCreated = new Intl.DateTimeFormat('id-ID', { day: '2-digit', month: 'long', year: 'numeric' }).format(new Date());
const newInput = ref({ title: '', priority: 2, tasks: [], deadline: new Date(), created: new Date() })
const minDate = new Date();
const toDo = ref([
    {
        id: 0, name: "To Do", todoTasks: []
    },
    {
        id: 1, name: "On Progress", todoTasks: []
    },
    {
        id: 2, name: "Done", todoTasks: []
    }
])
const modalValue = ref(false);
const handleTaskClick = (payload) => {
    modalValue.value = payload;
};

const toggleTask = (task) => {
    task.isDone = !task.isDone;
};

const startAddTask = () => {
    addingToDo.value = true;
    nextTick(() => {
        newTodoInput.value.focus();
    });
};
const addTask = () => {
    if (newTodoName.value.trim()) {
        newInput.value.tasks.push({ name: newTodoName.value, isDone: false });
        newTodoName.value = '';
        addingToDo.value = false;
    }
};
const deleteTask = (index) => {
    newInput.value.tasks.splice(index, 1);
};

const cancelAddTask = () => {
    addingToDo.value = false;
    newTodoName.value = '';
};

const setPriority = (input) => {
    newInput.value.priority = input
}

const handleTitle = (event) => {
    newInput.value.title = event.target.value;
}

const addTodo = (input) => {
    if (newInput.value.tasks.length == 0 || !newInput.value.title || newInput.value.title.trim() == "" || !newInput.value.deadline || !newInput.value.priority) {
        return
    }
    let category = 0

    if (newInput.value.tasks.every(t => t.isDone)) {
        category = 1
    } else if (newInput.value.tasks.some(t => t.isDone)) {
        category = 2
    } else {
        category = 0
    }

    const generateRandomId = () => {
        return '_' + Math.random().toString(36).substr(2, 9);
    };

    toDo.value[category].todoTasks.push({ id: generateRandomId(), name: newInput.value.title, priority: newInput.value.priority, created: new Date(), deadline: newInput.value.deadline, tasks: [...newInput.value.tasks] })
    newInput.value = { title: '', priority: 2, tasks: [], deadline: new Date(), created: new Date() };
    modalValue.value = false
}

const updatedToDo = computed(() => {
    const newToDo = { id: 0, name: "To Do", todoTasks: [] };
    const onProgress = { id: 1, name: "On Progress", todoTasks: [] };
    const done = { id: 2, name: "Done", todoTasks: [] };

    toDo.value.forEach((item) => {
        item.todoTasks.forEach((task) => {
            const allDone = task.tasks.every((subtask) => subtask.isDone);
            const someDone = task.tasks.some((subtask) => subtask.isDone);

            if (allDone) {
                done.todoTasks.push(task);
            } else if (someDone) {
                onProgress.todoTasks.push(task);
            } else {
                newToDo.todoTasks.push(task);
            }
        });
    });


    const clonedData = JSON.parse(JSON.stringify([newToDo, onProgress, done]));

    localStorage.setItem("todoMemory", JSON.stringify(clonedData));

    return [newToDo, onProgress, done];
});


const modalTop = computed(() => {
    return modalValue.value ? '0%' : '-100%';
});

const deleteCardTask = ({ id, newTask }) => {
    toDo.value = toDo.value.map((list, index) =>
        index === id ? { ...list, todoTasks: [...newTask.map(t => ({ ...t }))] } : list
    );
}


onBeforeMount(() => {
    const todoMemory = localStorage.getItem("todoMemory");
    let parsedToDoMemory;

    if (todoMemory) {
        try {
            parsedToDoMemory = JSON.parse(todoMemory);
        } catch (error) {
            console.error("Error parsing JSON from localStorage:", error);
            parsedToDoMemory = null;
        }
    }

    if (!parsedToDoMemory || parsedToDoMemory.length === 0) {
        const initialData = [
            { id: 0, name: "To Do", todoTasks: [] },
            { id: 1, name: "On Progress", todoTasks: [] },
            { id: 2, name: "Done", todoTasks: [] }
        ];
        localStorage.setItem("todoMemory", JSON.stringify(initialData));
        toDo.value = initialData;
    } else {
        toDo.value = parsedToDoMemory.map(t => ({
            ...t,
            todoTasks: t.todoTasks.map(task => ({ ...task }))
        }));
    }
});



</script>

<template>
    <div class="container">
        <div :class="{ blocker: modalValue, 'custom-hide': !modalValue }"></div>
        <div class="custom-modal-container" :style="{ top: modalTop, transition: '1s' }">
            <div class="custom-modal">
                <div class="custom-modal-header">
                    <div class="custom-modal-title">
                        <h3>Add To Do</h3>
                    </div>
                    <div class="close-modal">
                        <i class="far fa-times-circle fa-lg" style="color: red; cursor: pointer;"
                            @click.stop="handleTaskClick(false)"></i>
                    </div>
                    <div class="todo-cards-risk">
                        <div class="todo-title-input">
                            <p>To Do Title : &nbsp;</p>
                            <input type="text" name="todo-title" @input="handleTitle" id="todo-title">
                        </div>
                        <div class="todo-priority-input">
                            <span class="priority">Priority :&nbsp;</span>
                            <button :class="['low', { selected: newInput.priority == 1 }]"
                                @click="setPriority(1)">Low</button>
                            <button :class="['medium', { selected: newInput.priority == 2 }]"
                                @click="setPriority(2)">Medium</button>
                            <button :class="['high', { selected: newInput.priority == 3 }]"
                                @click="setPriority(3)">High</button>
                        </div>
                    </div>
                </div>
                <div class="custom-modal-content">
                    <div v-for="(task, index) in newInput.tasks" :key="index" class="todo-cards-task-item fix-task"
                        :class="{ done: task.isDone }" @click="toggleTask(task)">
                        <div>
                            <input type="checkbox" v-model="task.isDone" @click.stop />
                            {{ task.name }}
                        </div>
                        <div>
                            <i v-if="task.isDone" class="fas fa-check-circle" style="color: #63E6BE;"></i>
                            <i v-else class="fas fa-trash-alt" @click.stop="deleteTask(index)"></i>
                        </div>
                    </div>
                    <div v-if="addingToDo" class="todo-cards-task-item">

                        <i class="fas fa-plus"></i>
                        <input ref="newTodoInput" type="text" v-model="newTodoName" @keyup.enter="addTask"
                            @blur="cancelAddTask" placeholder="New task..." />
                    </div>

                    <div v-else class="todo-cards-task-item" @click="startAddTask">
                        <i class="fas fa-plus"></i>
                        Add Task
                    </div>
                </div>
                <div class="custom-modal-footer">
                    <div class="created">
                        <span class="created-date">Created At :&nbsp;</span>
                        <span>{{ newFullDateCreated }}</span>
                    </div>
                    <div class="deadline">
                        <span>Deadline :&nbsp;</span>
                        <VueDatePicker class="dt-custom" :min-date="minDate" :dark="true" :teleport="true"
                            v-model="newInput.deadline">
                        </VueDatePicker>
                    </div>
                    <div class="custom-modal-buttons">
                        <button class="cancel" @click.stop="handleTaskClick(false)">Cancel</button>
                        <button @click="addTodo" class="save">Save</button>
                    </div>
                </div>
            </div>
        </div>

        <TodoContainer v-for="item in updatedToDo" :key="item.id" :id="item.id" :name="item.name"
            :todoTasks="item.todoTasks" @taskClicked="handleTaskClick" @deleteCardTask="deleteCardTask" />

    </div>
</template>



<style>
#todo-title {
    margin-right: 20px;
    height: 100%;
    width: 70%;
}

.container {
    width: 100vw;
    height: 100vh;
    background-color: #ffffff;
    display: flex;
    flex-wrap: nowrap;
    padding: 20px;
    position: relative;
}

.blocker {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    background-color: #00000048;
    z-index: 4;
}

.custom-hide {
    display: none !important;
}

.custom-modal-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 5;
}

.custom-modal {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    width: 90%;
    height: 90%;
    background-color: #ffffff;
    border-radius: 10px;
    transition: 1s;
    padding: 20px 20px 0px 20px;
}

.custom-modal-header {
    position: relative;
    text-align: center;
    width: 100%;
}

.custom-modal-title {
    margin-bottom: 20px;
}

.custom-modal-title>h3 {
    font-size: 28px;
    font-weight: bold;
}

.custom-modal-content {
    width: 100%;
}

.custom-modal-footer {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: end;
    padding: 20px;
}

.custom-modal-footer>div {
    width: 33%;
}

.custom-modal-buttons {
    display: flex;
    justify-content: end;
}

.custom-modal-buttons>button {
    margin: 0 10px;
    padding: 10px;
    border: none;
    background: none;
    border-radius: 5px;

}

button.cancel {
    background-color: #D9534F;
    color: #ffffff;
    cursor: pointer;
}

button.save {
    background-color: #428BCA;
    color: #ffffff;
    cursor: pointer;
}

.close-modal {
    position: absolute;
    top: 0px;
    right: 0px;
}

.todo-cards-task-item {
    width: 100%;
    padding: 20px 10px;
    background-color: #F5F5F5;
    margin: 5px 0;
    border-radius: 10px;
    color: #000000;
    cursor: pointer;
}

.todo-cards-task-item input[type="checkbox"],
.todo-cards-task-item i {
    margin-right: 10px;
}

.todo-cards-task-item input[type="text"] {
    border: none;
    outline: none;
    box-shadow: none;
    background-color: transparent;
    width: 90%;
    font-size: 14px;
    color: #000;
}

.todo-cards-task-item input[type="text"]::placeholder {
    color: #999;
}

.todo-cards-task-item.done {
    text-decoration: line-through;
    color: #999;
}

.fix-task {
    display: flex;
    justify-content: space-between;
    width: 100%;
}

.created {
    padding: 5px 10px;
    border-radius: 5px;
    background-color: #000000;
    color: #ffffff;
    transition: 1s;
}

.created.expanded>span {
    font-size: 20px;
    transition: 1s;
}

.created-date {
    padding: 5px 10px;
    border-radius: 5px;
    background-color: #000000;
    color: #ffffff;
    transition: 1s;
}

.deadline,
.deadline>.deadline-date {
    display: flex;
    align-items: center;
}

.deadline>span {
    white-space: nowrap;
    margin-right: 10px;
}

.todo-cards-risk {
    width: 100%;
    margin: 10px 0;
    display: flex;
    justify-content: start;
}

.todo-title-input,
.todo-priority-input {
    width: 50%;
    display: flex;
    align-items: center;
}

.todo-title-input {
    display: flex;
    align-items: center;
}

.todo-title-input>p {
    width: 30%;
    height: 100%;
    display: flex;
    align-items: center;
}

.low {
    background-color: #428BCA !important;
}

.medium {
    background-color: #FFA500 !important;
}

.high {
    background-color: #D9534F !important;
}

button.selected {
    color: #ffffff;
}

.low,
.medium,
.high {
    margin: 0 5px;
    padding: 5px 5px;
    border: none;
    background: none;
    border-radius: 5px;
    cursor: pointer;
}

@media (max-width: 950px) {
    .container {
        flex-wrap: wrap;
    }

    .custom-modal-title>h3 {
        font-size: 20px !important;
    }

    .todo-title-input>p {
        font-size: 12px;
        width: 30%;
    }

    #todo-title {
        margin-right: 10px;
        width: 70%;
    }

    .created-date {
        font-size: 12px
    }

    .created {
        display: flex;
        align-items: center;
    }

    .created>span {
        font-size: 12px
    }

    .created.expanded>span {
        font-size: 12px;
        transition: 1s;
    }

    .deadline {
        font-size: 12px
    }

    .priority {
        font-size: 12px;
    }

    .dt-custom input {
        font-size: 12px !important;
    }

    .cancel {
        font-size: 12px !important;
    }

    .save {
        font-size: 12px !important;
    }

    .blocker {
        position: fixed;
    }

    .custom-modal-container {
        position: fixed;
    }
}

@media (max-width: 860px) {
    .created>span {
        font-size: 8px;
    }

    .deadline {
        font-size: 8px;
    }

    .dt-custom input {
        font-size: 8px !important;
    }

    .cancel {
        font-size: 8px !important;
    }

    .save {
        font-size: 8px !important;
    }

}

@media (max-width: 500px) {
    .todo-title-input>p {
        width: 30%;
    }

    #todo-title {
        width: 70%;
    }

    .low {
        font-size: 10px;
        background-color: #428BCA !important;
    }

    .medium {
        font-size: 10px;
        background-color: #FFA500 !important;
    }

    .high {
        font-size: 10px;
        background-color: #D9534F !important;
    }

    .custom-modal-footer>div {
        width: 100%;
        margin: 10px;
    }

    .custom-modal-footer {
        display: flex;
        flex-wrap: wrap;
    }

    .todo-cards-risk>div {
        width: 100%;
        margin: 10px;
    }

    .todo-cards-risk {
        display: flex;
        flex-wrap: wrap;
    }

    .todo-cards-task-item {
        padding: 10px 5px;
        font-size: 10px;
    }
}
</style>