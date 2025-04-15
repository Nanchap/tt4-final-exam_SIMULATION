<script>
  import { tasks } from './stores/tasks';
  import { onMount } from 'svelte';

  let title = '';
  let description = '';
  let editingTask = null;

  onMount(async () => {
    const response = await fetch('http://localhost:5156/api/tasks');
    if (response.ok) {
      tasks.set(await response.json());
    }
  });

  async function addOrUpdateTask() {
    const taskData = {
      title,
      description,
      completed: false,
    };

    if (editingTask) {
      // UPDATE
      const response = await fetch(`http://localhost:5156/api/tasks/${editingTask.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ ...editingTask, ...taskData }),
      });

      if (response.ok) {
        const updated = await response.json();
        tasks.update((t) =>
          t.map((task) => (task.id === updated.id ? updated : task))
        );
        resetForm();
      }
    } else {
      // CREATE
      const response = await fetch('http://localhost:5156/api/tasks', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(taskData),
      });

      if (response.ok) {
        const newTask = await response.json();
        tasks.update((t) => [...t, newTask]);
        resetForm();
      }
    }
  }

  async function deleteTask(id) {
    const response = await fetch(`http://localhost:5156/api/tasks/${id}`, {
      method: 'DELETE',
    });

    if (response.ok) {
      tasks.update((t) => t.filter((task) => task.id !== id));
    }
  }

  function startEdit(task) {
    editingTask = task;
    title = task.title;
    description = task.description;
  }

  function resetForm() {
    editingTask = null;
    title = '';
    description = '';
  }
</script>

<main class="container mt-4">
  <h1 class="mb-4">Task Manager</h1>

  <form class="mb-4" on:submit|preventDefault={addOrUpdateTask}>
    <div class="mb-3">
      <input class="form-control" type="text" placeholder="Title" bind:value={title} required />
    </div>
    <div class="mb-3">
      <input class="form-control" type="text" placeholder="Description" bind:value={description} required />
    </div>
    <button class="btn btn-primary me-2" type="submit">
      {editingTask ? 'Update Task' : 'Add Task'}
    </button>
    {#if editingTask}
      <button class="btn btn-secondary" type="button" on:click={resetForm}>Cancel</button>
    {/if}
  </form>

  <ul class="list-group">
    {#each $tasks as task (task.id)}
      <li class="list-group-item d-flex justify-content-between align-items-center">
        <div>
          <strong>{task.title}</strong> — {task.description} 
          <span class="badge bg-{task.completed ? 'success' : 'warning'} ms-2">
            {task.completed ? 'Completed' : 'Pending'}
          </span>
        </div>
        <div>
          <button class="btn btn-sm btn-outline-primary me-2" on:click={() => startEdit(task)}>
            Edit
          </button>
          <button class="btn btn-sm btn-outline-danger" on:click={() => deleteTask(task.id)}>
            Delete
          </button>
        </div>
      </li>
    {/each}
  </ul>
</main>
