<script setup>
import Card from '@/components/Card.vue'
import { defineProps, defineEmits, ref, onMounted, onUnmounted } from 'vue'
const emit = defineEmits(['taskClicked', 'deleteCardTask']);
const props = defineProps({
    id: Number,
    name: String,
    todoTasks: Array
});

const sort = ref(false)
const sortMenu = ref(null);
const toggleSortMenu = () => {
    sort.value = !sort.value;
};
const newTaskClicked = () => {
    emit('taskClicked', true);
};

const handleClickOutside = (event) => {
    if (sortMenu.value && !sortMenu.value.contains(event.target)) {
        sort.value = false;
    }
};
const handleDeleteCard = (index) => {
    props.todoTasks.splice(index, 1)
    emit("deleteCardTask", { id: props.id, newTask: props.todoTasks })
    console.log(props.todoTasks)
}
const sortTodo = (input) => {
    sort.value = false;

    if (input === 1) {
        props.todoTasks.sort((a, b) => new Date(a.deadline) - new Date(b.deadline));
    } else if (input === 2) {
        props.todoTasks.sort((a, b) => a.priority - b.priority);
        props.todoTasks.reverse()
    }
    console.log(props.todoTasks)
};

onMounted(() => {
    document.addEventListener('click', handleClickOutside);
});

onUnmounted(() => {
    document.removeEventListener('click', handleClickOutside);
});
</script>
<template>
    <div class="todo-container">
        <div class="todo">
            <div class="todo-header">
                <h2>{{ props.name }}</h2>
                <div class="todo-header-icons">
                    <i v-if="id == 0" class="fas fa-plus-circle fa-lg" style="color: #63E6BE; cursor: pointer;"
                        @click.stop="newTaskClicked"></i>
                    <i @click.stop="toggleSortMenu" class="fas fa-ellipsis-h fa-lg" style="cursor: pointer;">
                    </i>
                    <div v-if="sort" ref="sortMenu" class="sort-menu">
                        <p @click.stop="sortTodo(1)">Sort By Deadline</p>
                        <p @click.stop="sortTodo(2)">Sort By Priority</p>
                    </div>
                </div>
            </div>
            <div class="todo-content">
                <Card v-for="(task, index) in props.todoTasks" :key="task.id || index" :tasks="task" :t_id="props.id"
                    :index="index" @deleteCard="handleDeleteCard" />
            </div>
        </div>
    </div>
</template>

<style scoped>
.todo-container {
    width: 33%;
    height: 100%;
    padding: 10px;
}

.todo {
    width: 100%;
    height: 100%;
    border: 1px solid #EAEFF0;
    border-radius: 20px;
}

.todo-header {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 20px;
}

.todo-header>h2 {
    font-weight: bold;
}

.todo-header-icons {
    display: flex;
    position: relative;
}

.todo-header-icons>i {
    margin: 0 10px;
}

.todo-content {
    width: 100%;
    max-height: 90%;
    overflow-y: auto;
    /* Retain vertical scrolling */
    padding: 15px 20px;
}

.sort-menu {
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 0;
    right: 0;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #000000;
    background-color: #ffffff;
    font-size: 14px;
    white-space: pre;
    z-index: 3;
}

.sort-menu>p {
    cursor: pointer;
    padding: 5px 3px;
    border-radius: 3px;
}

.sort-menu>p:hover {
    background-color: #000000;
    color: #ffffff;
}

@media (max-width: 950px) {
    .todo-container {
        width: 100%;
    }
}
</style>