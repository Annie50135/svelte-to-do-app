<script>
  import { onMount } from 'svelte';

  let tasks = $state([]);
  let newTask = $state('');
  let newCategory = $state('Personal');
  let newDate = $state('');
  let filter = $state('all');
  let darkMode = $state(false);
  let ready = $state(false);

  onMount(() => {
    try {
      const saved = localStorage.getItem('my-todo-tasks');
      if (saved) {
        const parsed = JSON.parse(saved);
        if (Array.isArray(parsed)) tasks = parsed;
      }

      darkMode = localStorage.getItem('todo-dark-mode') === 'true';
    } catch (error) {
      console.error('Loading error:', error);
    }

    ready = true;
  });

  $effect(() => {
    if (ready) {
      localStorage.setItem('my-todo-tasks', JSON.stringify(tasks));
      localStorage.setItem('todo-dark-mode', String(darkMode));
    }
  });

  function addTask() {
    if (!newTask.trim()) return;

    tasks.push({
      id: Date.now(),
      title: newTask.trim(),
      category: newCategory,
      dueDate: newDate,
      completed: false
    });

    newTask = '';
    newDate = '';
    newCategory = 'Personal';
  }

  function deleteTask(id) {
    tasks = tasks.filter(task => task.id !== id);
  }

  function toggleTask(id) {
    tasks = tasks.map(task =>
      task.id === id
        ? { ...task, completed: !task.completed }
        : task
    );
  }

  let completedCount = $derived(
    tasks.filter(task => task.completed).length
  );

  let progress = $derived(
    tasks.length
      ? Math.round((completedCount / tasks.length) * 100)
      : 0
  );

  let filteredTasks = $derived(
    tasks.filter(task => {
      if (filter === 'active') return !task.completed;
      if (filter === 'completed') return task.completed;
      return true;
    })
  );
</script>

<svelte:head>
  <title>My Task Dashboard</title>
  <meta name="description" content="Organize and manage your daily tasks." />
</svelte:head>

<main class:dark={darkMode}>
  <div class="dashboard">

    <header>
      <div>
        <p class="eyebrow">MY PERSONAL WORKSPACE</p>
        <h1>My Dashboard ✨</h1>
        <p class="subtitle">Make today count. One task at a time.</p>
      </div>

      <button class="theme-button" onclick={() => darkMode = !darkMode}>
        {darkMode ? '☀️ Light' : '🌙 Dark'}
      </button>
    </header>

    <section class="overview">
      <div class="stat-card">
        <span>Total tasks</span>
        <strong>{tasks.length}</strong>
      </div>

      <div class="stat-card">
        <span>Completed</span>
        <strong>{completedCount}</strong>
      </div>

      <div class="stat-card">
        <span>Remaining</span>
        <strong>{tasks.length - completedCount}</strong>
      </div>
    </section>

    <section class="progress-card">
      <div class="progress-heading">
        <strong>Your progress</strong>
        <strong>{progress}%</strong>
      </div>

      <div class="progress-track">
        <div class="progress-fill" style={`width: ${progress}%`}></div>
      </div>

      <p>Keep going! Every completed task is a win.</p>
    </section>

    <section class="task-card">
      <h2>Add a new task</h2>

      <form onsubmit={(event) => {
        event.preventDefault();
        addTask();
      }}>
        <label for="task">Task name</label>
        <input
          id="task"
          bind:value={newTask}
          placeholder="What do you need to do?"
          required
        />

        <div class="form-row">
          <div>
            <label for="category">Category</label>
            <select id="category" bind:value={newCategory}>
              <option>Personal</option>
              <option>Work</option>
              <option>School</option>
              <option>Shopping</option>
              <option>Other</option>
            </select>
          </div>

          <div>
            <label for="dueDate">Due date</label>
            <input id="dueDate" type="date" bind:value={newDate} />
          </div>
        </div>

        <button class="add-button" type="submit">
          + Add Task
        </button>
      </form>
    </section>

    <section class="task-list">
      <div class="list-heading">
        <h2>My tasks</h2>
        <span>{filteredTasks.length} shown</span>
      </div>

      <div class="filters">
        <button class:chosen={filter === 'all'} onclick={() => filter = 'all'}>All</button>
        <button class:chosen={filter === 'active'} onclick={() => filter = 'active'}>Active</button>
        <button class:chosen={filter === 'completed'} onclick={() => filter = 'completed'}>Completed</button>
      </div>

      {#if filteredTasks.length === 0}
        <div class="empty">
          <div>📝</div>
          <p>No tasks here yet.</p>
          <span>Add a task to get started!</span>
        </div>
      {:else}
        {#each filteredTasks as task (task.id)}
          <article class="task-item">
            <label class="task-label">
              <input
                type="checkbox"
                checked={task.completed}
                onchange={() => toggleTask(task.id)}
              />

              <span class:done={task.completed}>
                {task.title}
                <small>
                  {task.category || 'Personal'}
                  {#if task.dueDate}
                    · Due: {task.dueDate}
                  {/if}
                </small>
              </span>
            </label>

            <button class="delete-button" onclick={() => deleteTask(task.id)}>
              Delete
            </button>
          </article>
        {/each}
      {/if}
    </section>

    <footer>Built with ❤️ using Svelte</footer>
  </div>
</main>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(body) {
    margin: 0;
    font-family: Arial, sans-serif;
  }

  main {
    min-height: 100vh;
    padding: 35px 18px;
    background: #f2f3fb;
    color: #25243a;
    transition: background 0.2s, color 0.2s;
  }

  main.dark {
    background: #171827;
    color: #f3f3fa;
  }

  .dashboard {
    max-width: 760px;
    margin: auto;
  }

  header, .progress-heading, .list-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 15px;
  }

  .eyebrow {
    color: #7265d8;
    font-size: 11px;
    font-weight: bold;
    letter-spacing: 2px;
  }

  h1 {
    margin: 8px 0;
    font-size: 32px;
  }

  h2 {
    font-size: 19px;
  }

  .subtitle, .progress-card p {
    color: #85849a;
  }

  .theme-button, .filters button {
    padding: 11px 15px;
    border: 1px solid #deddf0;
    border-radius: 9px;
    background: white;
    color: #5546d8;
    cursor: pointer;
  }

  .dark .theme-button, .dark .filters button {
    background: #282a40;
    color: white;
    border-color: #41435b;
  }

  .overview {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 13px;
    margin: 25px 0;
  }

  .stat-card, .progress-card, .task-card, .task-list {
    background: white;
    border-radius: 14px;
    padding: 20px;
    box-shadow: 0 5px 20px #2420440b;
  }

  .dark .stat-card, .dark .progress-card,
  .dark .task-card, .dark .task-list {
    background: #22243a;
    color: #f3f3fa;
  }

  .stat-card span {
    display: block;
    color: #85849a;
    font-size: 13px;
  }

  .stat-card strong {
    display: block;
    font-size: 27px;
    margin-top: 10px;
    color: #6555d9;
  }

  .progress-card {
    margin-bottom: 20px;
  }

  .progress-track {
    height: 10px;
    border-radius: 20px;
    background: #e8e6f5;
    overflow: hidden;
    margin-top: 15px;
  }

  .progress-fill {
    height: 100%;
    background: #6555d9;
    border-radius: 20px;
    transition: width 0.3s;
  }

  .progress-card p {
    font-size: 13px;
  }

  form label {
    display: block;
    margin: 13px 0 7px;
    font-size: 13px;
    font-weight: bold;
  }

  form input, form select {
    width: 100%;
    min-width: 0;
    padding: 12px;
    border: 1px solid #deddf0;
    border-radius: 8px;
    background: white;
    color: #25243a;
  }

  .dark form input, .dark form select {
    background: #303249;
    color: white;
    border-color: #41435b;
  }

  .form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 13px;
  }

  button {
    cursor: pointer;
  }

  .add-button {
    width: 100%;
    margin-top: 18px;
    padding: 14px;
    background: #6555d9;
    color: white;
    border: 0;
    border-radius: 9px;
    font-weight: bold;
  }

  .task-list {
    margin-top: 20px;
  }

  .list-heading span {
    color: #85849a;
    font-size: 13px;
  }

  .filters {
    display: flex;
    gap: 8px;
    margin: 15px 0;
  }

  .filters button {
    padding: 9px 14px;
  }

  .filters button.chosen {
    background: #6555d9;
    color: white;
    border-color: #6555d9;
  }

  .task-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    padding: 15px 0;
    border-bottom: 1px solid #eee;
  }

  .dark .task-item {
    border-color: #41435b;
  }

  .task-label {
    display: flex;
    align-items: center;
    gap: 12px;
    min-width: 0;
    overflow-wrap: anywhere;
  }

  .task-label input {
    width: 17px;
    height: 17px;
    accent-color: #6555d9;
    flex-shrink: 0;
  }

  .task-label small {
    display: block;
    color: #85849a;
    margin-top: 5px;
  }

  .done {
    text-decoration: line-through;
    color: #999;
  }

  .delete-button {
    padding: 8px 11px;
    border: 0;
    border-radius: 7px;
    background: #ffe9e9;
    color: #cf4242;
    flex-shrink: 0;
  }

  .empty {
    text-align: center;
    padding: 30px 10px;
    color: #85849a;
  }

  .empty div {
    font-size: 35px;
  }

  footer {
    text-align: center;
    padding: 25px;
    color: #85849a;
    font-size: 13px;
  }

  @media (max-width: 480px) {
    h1 {
      font-size: 25px;
    }

    .overview {
      gap: 8px;
    }

    .stat-card {
      padding: 13px;
    }

    .stat-card strong {
      font-size: 22px;
    }

    .form-row {
      grid-template-columns: 1fr;
    }
  }
</style>