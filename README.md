# Vue.js To-Do List Application

## Overview

This application is a to-do list management system built with Vue.js. It allows users to add, manage, and organize tasks into different categories such as "To Do," "On Progress," and "Done." It includes features for adding new tasks with detailed attributes, sorting tasks, and ensuring data persistence across sessions.

## Key Features

Task Management : Users can add new tasks, set their priority, and define deadlines.

- Task Organization : Tasks are categorized into different sections like "To Do," "On Progress," and "Done," and can be moved between these categories based on their completion status.
- Data Persistence : The application uses localStorage to save the state of the to-do list. This ensures that user data persists across sessions, so tasks and categories remain consistent even after the browser is closed or refreshed.
- Date Formatting : Dates are formatted according to the Indonesian locale for user convenience.

## How It Works

- Adding Tasks : Users can open a modal to input new tasks, including title, priority, and deadline. Tasks are then added to the appropriate category.
- Managing Tasks : Tasks can be marked as completed, which automatically moves them to the "Done" category, or they can be deleted if no longer needed.
- Persistence with LocalStorage : Upon initialization, the application loads the to-do list from localStorage. Any changes made to the tasks or categories are saved back to localStorage, ensuring data is retained across sessions.

## Main Data Structure

The main data used in the application consists of categories and tasks. Here’s a breakdown of the data structure :

```js
;[
	{
		id: 0,
		name: "To Do",
		todoTasks: [],
	},
	{
		id: 1,
		name: "On Progress",
		todoTasks: [],
	},
	{
		id: 2,
		name: "Done",
		todoTasks: [],
	},
]
```

- id : A unique identifier for the category.
- name : The name of the category (e.g., "To Do," "On Progress," "Done").
- todoTasks : An array containing tasks assigned to this category.
  **Task**

```js
{
    "id" : generateRandomId(),
    "name" : newInput.value.title,
    "priority" : newInput.value.priority,
    "created" : new Date(),
    "deadline" : newInput.value.deadline,
    "tasks" : [...newInput.value.tasks]
}
```

- id : A unique identifier for the task, generated using generateRandomId().
- name : The title of the task, provided by the user.
- priority : The priority level of the task (e.g., Low, Medium, High), set by the user.
- created : The date and time when the task was created.
- deadline : The deadline for the task, as set by the user.
- tasks : An array of subtasks associated with the main task.

## Components

### HomeView

**Description** : The HomeView component serves as the main view of the application. It manages the state for adding and editing tasks and displays different to-do categories using the TodoContainer component.

**Key Features** :
Task Addition : Provides a modal to add new tasks with title, priority, and deadline.
Task Management : Handles tasks by moving them between categories based on their completion status.
Persistence : Stores the to-do list in localStorage to persist data between sessions.
Date Formatting : Formats dates in Indonesian locale.

**Function** :

- Reactive Variables :

  - addingToDo : Boolean flag to show/hide the modal for adding tasks.
  - newTodoInput : Reference to the input field for new Todo List.
  - newFullDateCreated : Formatted string of the current date reference to input created date.
  - newInput : Object holding details for the new task being created.
  - minDate : Minimum selectable date for the deadline picker which is current date.
  - toDo : Array of categories Todo (To Do, On Progress, Done).
  - modalValue : Boolean flag to control modal visibility.

- Methods :

  - handleTaskClick(payload) : Toggles modal visibility.
  - toggleTask(task) : Marks a task as done or not done.
  - startAddTask() : add a new task.
  - addTask() : Adds a new task to the Todo.
  - deleteTask(index) : Removes a task from the Todo.
  - cancelAddTask() : Cancels the addition of a new task.
  - setPriority(input) : Sets the priority of the new Todo.
  - handleTitle(event) : Updates the title of the new task.
  - addTodo(input) : Adds a new to-do Todo item to the appropriate category.
  - updatedToDo : Computes categorized tasks based on completion status.
  - modalTop : Computes the position of the modal.
  - deleteCardTask({ id, newTask }) : Updates the tasks in a specific category.

- Lifecycle Hooks :
  - onBeforeMount() : Initializes the to-do list from localStorage or sets default values.

### TodoContainer

**Description** : The TodoContainer component is responsible for rendering and managing a collection of Card components, each representing a to-do list category.

**Key Features** :

- Dynamic Rendering : Displays a list of Card components dynamically based on the tasks and categories.
- Categorization : Groups tasks into categories and provides visual separation.
- Task Removal : Allows for the removal of entire categories.
- Task Management : Displays tasks and allows sorting by deadline or priority.
- Responsive Design: Adjusts layout based on screen size
  **Function** :

- Reactive Variables :
  - sort : Boolean flag to toggle the visibility of the sort menu.
  - sortMenu : Reference to the sort menu element for detecting clicks outside of it.
- Methods:
  - toggleSortMenu: Toggles the visibility of the sort menu.
  - newTaskClicked: Emits the taskClicked event to add new Todo list, which opens the task addition modal.
  - handleClickOutside(event): Closes the sort menu if a click is detected outside of it.
  - handleDeleteCard(index): Deletes a task from the todoTasks array and emits the deleteCardTask event.
  - sortTodo(input): Sorts tasks based on the selected criterion (input can be 1 for deadline or 2 for priority).
- Lifecyle Hooks :
  - Mounted: Adds an event listener for clicks to handle clicks outside of the sort menu.
  - Unmounted: Removes the event listener for clicks when the component is destroyed.

### Card

**Description** : The Card component represents a single to-do list category, allowing for detailed task management within that category.

**Key Features** :

- Expandable View : Supports expanding and collapsing to show more details and tasks.
- Task Management : Provides functionality to add, toggle, and delete tasks within the card.
- Dynamic Priority Display : Shows priority levels with color-coded backgrounds.
- Date Formatting : Displays creation and deadline dates in various formats.
- Responsive Design : Adjusts layout and styling based on screen size.

**Function** :

- Reactive Variables :

  - isExpanded : Boolean flag to indicate whether the card is displayed in detail.
  - tasks : Ref to the list of tasks within the card.
  - temporaryTasks : Ref to a temporary list of tasks used for expansion transitions.
  - card : Ref to the card DOM element for size and position calculations.
  - priority : Ref to the priority level of the card, formatted as a string.
  - cardHeader : Ref to the card header DOM element.
  - initialWidth : Ref to the initial width, height, top, and left position of the card.
  - totalTasks : Ref to the total number of tasks in the card.
  - created : Ref to the creation date of the card.
  - now : Holds the current date for comparison with deadline.
  - deadlineDate : Ref to the deadline date of the card.
  - shortDateCreated : Computed property for formatted short creation date, displayed when expanded is false.
  - fullDateCreated : Computed property for formatted full creation date, displayed when expanded is true.
  - shortDateDeadline : Computed property for formatted short deadline date, displayed when expanded is false.
  - fullDateDeadline : Computed property for formatted full deadline date, displayed when expanded is true.
  - addingTask : Boolean flag to indicate if a new task is being added.
  - newTaskName : Ref to the name of the new task being added.
  - newTaskInput : Ref to the input element for adding new tasks.

- Methods :

  - toggleExpand({ from }): Handles the expansion and collapsing of the card, including animation.
  - toggleTask(task): Toggles the completion status of a task.
  - startAddTask: Prepares the todo list for adding a new task.
  - addTask: Adds a new task to the todo list.
  - deleteTask(index): Deletes a task from the todo list by its index.
  - cancelAddTask: Cancels the task addition process.
  - priorityAssignment: Sets the priority string based on the priority number.
  - daysBetween(date1, date2): Calculates the number of days between two dates.

- Computed Properties:

  - daysDiff: Computes the difference in days between the current date and the deadline.
  - dateColor: Determines the background color based on the deadline's proximity.
  - divStyle: Provides styling for the card based on the deadline's color.

- Watchers :

  - props.tasks.deadline : Updates the deadline date when the prop changes.
  - props.tasks.created : Updates the creation date when the prop changes.
  - props.tasks.priority : Updates the priority string when the prop changes.

- Lifecycle Hooks:
  - onMounted: Initializes the card's size and position and assigns the priority string.

## Advantages:

- Fast Fetching : Data is stored in internal browser memory, making subsequent fetches fast.
- Lightweight : The component is very light, ensuring quick load times and efficient performance.
- Simplicity : Easy to use with a straightforward interface.
- No Registration Needed : Users can start using the application immediately without the need for registration.
- Responsive Design : Adapts to various screen sizes, providing a seamless experience on different devices.
- Data Privacy : No need to worry about leaked data since data is stored on the user's own device.

# Steps to Install and Run

## Clone the Repository

Start by cloning the repository that contains the TodoContainer component :

```bash
git clone <repository-url>
```

## Navigate to the Project Directory

Change directory to the cloned project folder :

```bash
cd <project-directory>
```

### Install Dependencies

Install the necessary dependencies using npm :

<pre style="display: none;"><code>npm install</code></pre>

<button style="position: absolute; right: 0; top: 0%;" onclick="navigator.clipboard.writeText('npm install')">Copy</button>

<pre><code>npm install</code></pre>

Start the development server to see the component in action :

<pre style="display: none;"><code>npm run dev</code></pre>

<button style="position: absolute; right: 0; top: 0%;" onclick="navigator.clipboard.writeText('npm run dev')">Copy</button>

<pre><code>npm run dev</code></pre>

This command will run the project and start a local development server. By default, it will be available at http://localhost:9199.
You can change the port in vite.config.json

To create a production build of your project, run:

<pre style="display: none;"><code>npm run build</code></pre>

<button style="position: absolute; right: 0; top: 0%;" onclick="navigator.clipboard.writeText('npm run build')">Copy</button>

<pre><code>npm run build</code></pre>

## Screenshots

### Home

**Description** : The Home screen displays all your tasks categorized by their completion status. It allows users to quickly add, view, and manage their tasks. It has three categories: Todo, On Progress, and Done.

![Homescreen](src\assets\picture\todo-1.png)

### Todo Modal

**Description** : You can input a title, set priority, add multiple tasks, and set a deadline. The deadline background color changes as it gets closer to the deadline.

![Todomodal](src\assets\picture\todo-3.png)

### Card Preview

**Description** : Hovering over a card triggers a pop-out animation. Clicking on the card takes you to its detail view.
![Cardpreview](src\assets\picture\todo-4.png)

### Sort Todo

**Description** : Sort tasks by the nearest deadline or highest priority.
![Sort](src\assets\picture\todo-5.png)

### Detail Todo

**Description** : View detailed task information and mark tasks as complete. The task's status will update automatically:

- If no tasks are completed, it remains in the "To Do" category.
- If some tasks are completed, it moves to the "On Progress" category.
- If all tasks are completed, it moves to the "Done" category.

The background color of the deadline changes based on how close the deadline is :

- Red if there are 1 or fewer days remaining
- Green if there are 5 or fewer days remaining
- Blue if there are more than 5 days remaining
  ![Detail](src\assets\picture\todo-6.png)

### Responsive

**Description** : The view will adjusted accordingly based on screen
![Responsive](src\assets\picture\todo-7.png)
![Responsive](src\assets\picture\todo-8.png)

## Staytuned for next update :)