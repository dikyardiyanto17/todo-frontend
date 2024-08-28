<script setup>
import { ref, computed, watch, onMounted, defineEmits, defineProps } from 'vue';

const emit = defineEmits(['deleteCard']);
const props = defineProps({
    tasks: Object,
    t_id: Number,
    index: Number
});

const deletedCard = () => {
    emit('deleteCard', props.index)
}

const isExpanded = ref(false);
const tasks = ref([...props.tasks.tasks]);
const temporaryTasks = ref([...tasks.value]);
const card = ref(null);
const priority = ref("");
const cardHeader = ref(null);
let initialWidth = ref({ width: 0, height: 0, top: 0, left: 0 });
let totalTasks = ref(0);
const created = ref(new Date(props.tasks.created));
const now = new Date();
const deadlineDate = ref(new Date(props.tasks.deadline));
const shortDateCreated = computed(() => new Intl.DateTimeFormat('id-ID', { day: '2-digit', month: 'short' }).format(new Date(created.value)));
const fullDateCreated = computed(() => new Intl.DateTimeFormat('id-ID', { day: '2-digit', month: 'long', year: 'numeric' }).format(new Date(created.value)));
const shortDateDeadline = computed(() => new Intl.DateTimeFormat('id-ID', { day: '2-digit', month: 'short' }).format(deadlineDate.value));
const fullDateDeadline = computed(() => new Intl.DateTimeFormat('id-ID', { day: '2-digit', month: 'long', year: 'numeric' }).format(deadlineDate.value));
const addingTask = ref(false);
const newTaskName = ref('');
const newTaskInput = ref(null);

const toggleExpand = async ({ from }) => {
    if (isExpanded.value && from === "card") return;

    if (isExpanded.value) {
        props.tasks.tasks = temporaryTasks.value.map(task => ({ ...task }))
        card.value.style.width = `${initialWidth.value.width}px`;
        card.value.style.height = `${initialWidth.value.height}px`;
        card.value.style.top = `${initialWidth.value.top}px`;
        card.value.style.left = `${initialWidth.value.left}px`;
        card.value.style.transform = '';
        card.value.style.position = 'absolute';
        isExpanded.value = false;
        setTimeout(() => {
            card.value.style.width = '';
            card.value.style.height = '';
            card.value.style.top = '';
            card.value.style.left = '';
            card.value.style.transform = '';
            card.value.style.transition = 'all 1s ease';
            card.value.style.position = '';
        }, 1000);
    } else {
        temporaryTasks.value = tasks.value.map(task => ({ ...task }))
        const rect = card.value.getBoundingClientRect();
        const { width, height, top, left } = rect;
        card.value.style.width = `${width}px`;
        card.value.style.height = `${height}px`;
        card.value.style.top = `${top}px`;
        card.value.style.left = `${left}px`;
        card.value.style.transform = 'translateY(0) translateX(0)';
        card.value.style.transition = 'all 1s ease';
        initialWidth.value = { width, height, top, left };

        void card.value.offsetWidth;

        card.value.style.width = '100vw';
        card.value.style.height = '100vh';
        card.value.style.top = '0';
        card.value.style.left = '0';
        card.value.style.transform = '';
        isExpanded.value = true;
    }
};

const toggleTask = (task) => {
    task.isDone = !task.isDone;
};

const startAddTask = () => {
    addingTask.value = true;
    nextTick(() => {
        newTaskInput.value.focus();
    });
};

const addTask = () => {
    if (newTaskName.value.trim()) {
        tasks.value.push({ name: newTaskName.value, isDone: false });
        newTaskName.value = '';
        addingTask.value = false;
    }
};

const deleteTask = (index) => {
    tasks.value.splice(index, 1);
    temporaryTasks.value = tasks.value.map(task => ({ ...task }))
};

const cancelAddTask = () => {
    addingTask.value = false;
    newTaskName.value = '';
};

const priorityAssignment = () => {
    switch (props.tasks.priority) {
        case 1:
            priority.value = "Low";
            break;
        case 2:
            priority.value = "Medium";
            break;
        case 3:
            priority.value = "Urgent";
            break;
        default:
            priority.value = "Unknown";
            break;
    }
};

const daysBetween = (date1, date2) => {
    const date1Timestamp = new Date(date1).getTime();
    const date2Timestamp = new Date(date2).getTime();
    const diffTime = Math.abs(date2Timestamp - date1Timestamp);
    return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
};

const daysDiff = computed(() => daysBetween(now, deadlineDate.value));

const dateColor = computed(() => {
    if (daysDiff.value <= 1) {
        return 'red';
    } else if (daysDiff.value <= 5) {
        return 'green';
    } else if (daysDiff.value > 5) {
        return "blue";
    } else {
        return 'black';
    }
});

const divStyle = computed(() => ({
    width: '33%',
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: dateColor.value
}));


watch(() => props.tasks.deadline, (newValue) => {
    deadlineDate.value = new Date(newValue);
});

watch(() => props.tasks.created, (newValue) => {
    created.value = new Date(newValue);
});
watch(() => props.tasks.priority, () => {
    priorityAssignment();
});

onMounted(() => {
    if (card.value) {
        const rect = card.value.getBoundingClientRect();
        initialWidth.value = { width: rect.width, height: rect.height, top: rect.y, left: rect.left };
    }
    totalTasks.value = tasks.value.length;
    priorityAssignment();
});
</script>



<template>
    <div ref="card" :class="['todo-cards', { expanded: isExpanded }]" @click.stop="toggleExpand({ from: 'card' })">
        <div :class="isExpanded ? 'close-button' : 'custom-hide'">
            <i class="far fa-times-circle fa-lg" style="color: red; cursor: pointer;"
                @click.stop="toggleExpand({ from: 'expanded' })"></i>
        </div>
        <div ref="cardHeader" :class="['todo-cards-header', { expanded: isExpanded }]">
            <div :class="['todo-cards-header-icon', { expanded: isExpanded }]">
                <i v-if="t_id == 2" class="fas fa-clipboard-check" style="color: #63E6BE;"></i>
                <i v-else class="fas fa-hourglass-half" style="color: #74C0FC;"></i>
            </div>
            <div :class="['todo-cards-header-title', { expanded: isExpanded }]">
                <h3>{{ props.tasks.name }}</h3>
            </div>
            <div :class="['todo-cards-remove', { expanded: isExpanded }]">
                <i v-if="t_id == 2" class="fas fa-trash-alt" @click.stop="deletedCard"></i>
            </div>
        </div>
        <div :class="['todo-cards-risk', { expanded: isExpanded }]">
            <span :class="isExpanded ? 'priority' : 'custom-hide'">Priority :&nbsp;</span><span
                :class="priority.toLowerCase()">{{ priority }}</span>
        </div>
        <div :class="isExpanded ? 'todo-cards-task' : 'custom-hide'">
            <div v-for="(task, index) in temporaryTasks" :key="index" class="todo-cards-task-item fix-task"
                :class="{ done: task.isDone }" @click.stop="toggleTask(task)">
                <div>
                    <input type="checkbox" v-model="task.isDone" @click.stop />
                    {{ task.name }}
                </div>
                <div>
                    <i v-if="task.isDone" class="fas fa-check-circle" style="color: #63E6BE;"></i>
                    <i v-else class="fas fa-trash-alt" @click.stop="deleteTask(index)"></i>
                </div>
            </div>
        </div>
        <div v-if="addingTask" class="todo-cards-task-item">

            <i class="fas fa-plus"></i>
            <input ref="newTaskInput" type="text" v-model="newTaskName" @keyup.enter="addTask" @blur="cancelAddTask"
                placeholder="New task..." />
        </div>

        <div v-else-if="isExpanded && !addingTask" class="todo-cards-task-item" @click="startAddTask">
            <i class="fas fa-plus"></i>
            Add Task
        </div>

        <div class="todo-cards-content">
            <div style="width: 33%; display: flex; justify-content: start; align-items: center;"
                :class="['created', { expanded: isExpanded }]">
                <span :class="isExpanded ? 'created-date' : 'custom-hide'">Created At :&nbsp;</span>
                <span>{{ isExpanded ? fullDateCreated : shortDateCreated }}</span>
            </div>
            <div :style="divStyle" :class="['deadline', { expanded: isExpanded }]">
                <span :class="isExpanded ? 'deadline-date' : 'custom-hide'">Deadline
                    :&nbsp;</span>
                <span>{{ isExpanded ? fullDateDeadline : shortDateDeadline }}</span>
            </div>
            <div style="width: 33%; display: flex; justify-content: end; align-items: center;">
                <p style="cursor: pointer;"><span>{{ totalTasks }}&nbsp;&nbsp;&nbsp;</span><i
                        class="fas fa-stream lg"></i>
                </p>
            </div>
        </div>
    </div>
</template>


<style scoped>
.close-button {
    position: absolute;
    top: 20px;
    right: 20px;
}

.fix-task {
    display: flex;
    justify-content: space-between;
    width: 100%;
}

.low {
    background-color: green !important;
    margin-right: 10px;
    cursor: pointer;
    transition: 1s;
}

.medium {
    background-color: orange !important;
    margin-right: 10px;
    cursor: pointer;
    transition: 1s;
}

.high {
    background-color: red !important;
    margin-right: 10px;
    cursor: pointer;
    transition: 1s;
}

h3 {
    font-size: 14px;
}

p>span {
    font-size: 10px;
}

.priority {
    background-color: rgba(63, 63, 63, 0) !important;
    color: black !important;
}

.custom-hide {
    display: none !important;
    transition: 1s;
}

.todo-cards {
    border: 1px solid #EAEFF0;
    border-radius: 10px;
    padding: 20px;
    margin: 10px 0;
    position: relative;
    z-index: 2;
    transition: transform 0.3s ease, box-shadow 0.3s ease, width 1s ease, height 1s ease, top 1s ease, left 1s ease;
    background-color: #ffffff;
}

.todo-cards.expanded {
    position: absolute;
    z-index: 10;
    padding: 40px;
    background-color: #ffffff;
    border-radius: 0;
    margin: 0;
    transition: 1s;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
}

.todo-cards:not(.expanded):hover {
    transform: translateY(10px) translateX(10px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
    z-index: 3;
}

.todo-cards:not(.expanded) {
    cursor: pointer;
}

.todo-cards-header {
    width: 100%;
    display: flex;
    align-items: center;
}

.todo-cards-header.expanded {
    width: 100%;
    display: flex;
    justify-content: center;
    transition: 1s;
}

.todo-cards-header-icon {
    width: 15%;
    transition: 1s;
}

.todo-cards-header-icon.expanded {
    width: 0%;
    display: none;
    transition: 1s;
}

.todo-cards-header-title {
    width: 70%;
    transition: 1s;
}

.todo-cards-remove {
    display: flex;
    justify-content: end;
    width: 15%;
    transition: 1s;
}

.todo-cards-remove.expanded {
    width: 0%;
}

.todo-cards-header-title.expanded {
    width: 100%;
    transition: 1s;
}

.todo-cards-header-title.expanded>h3 {
    transition: 1s;
    font-size: 30px;
    font-weight: bold;
    text-align: center;
}

.todo-cards-risk {
    width: 100%;
    margin: 10px 0;
}

.todo-cards-risk>span {
    background-color: red;
    border-radius: 5px;
    padding: 5px 10px;
    font-size: 10px;
    color: #ffffff;
}

.todo-cards-risk.expanded>span {
    background-color: red;
    border-radius: 5px;
    padding: 5px 10px;
    font-size: 20px !important;
    color: #ffffff;
}

.todo-cards-task {
    width: 100%;
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    overflow: auto;
    transition: 1s;
    margin: 20px 0;
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

.todo-cards-content {
    width: 100%;
    display: flex;
    justify-content: space-between;
    margin: 20px 0 0px 0;
    align-items: center;
}

.created,
.deadline {
    padding: 5px 10px;
    border-radius: 5px;
    background-color: #000000;
    color: #ffffff;
    font-size: 14px;
    transition: 1s;
}

.created.expanded>span,
.deadline.epanded>span {
    font-size: 14px;
    transition: 1s;
}

.created-date,
.deadline-date {
    padding: 5px 10px;
    border-radius: 5px;
    color: #ffffff;
    font-size: 14px;
    transition: 1s;
}

@media (max-width: 950px) {
    .todo-cards.expanded {
        position: fixed;
    }

    .created-date {
        font-size: 12px;
    }

    .created {
        display: flex;
        align-items: center;
    }

    .created>span {
        font-size: 12px;
    }

    .created.expanded>span {
        font-size: 12px;
        transition: 1s;
    }

    .deadline {
        font-size: 12px;
    }

    .deadline-date {
        font-size: 12px;
    }


    .todo-cards-risk.expanded>span {
        font-size: 16px !important;
    }
}

@media (max-width: 550px) {
    .created-date {
        font-size: 8px;
    }

    .created {
        display: flex;
        align-items: center;
    }

    .created>span {
        font-size: 8px;
    }

    .created.expanded>span {
        font-size: 8px;
        transition: 1s;
    }

    .created.expanded {
        padding: 3px;
    }

    .deadline {
        font-size: 8px;
    }

    .deadline.expanded {
        padding: 3px;
    }

    .deadline-date {
        font-size: 8px;
        background-color: none !important;
    }

    .todo-cards-risk.expanded>span {
        font-size: 12px !important;
    }
}
</style>
