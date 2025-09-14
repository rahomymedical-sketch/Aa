<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>برنامج المهام المتطور برو</title>
  <style>
    :root {
      --bg-light: #f5f5f5;
      --bg-dark: #1e1e1e;
      --text-light: #000;
      --text-dark: #fff;
      --primary: #6c63ff;
      --secondary: #ff6f61;
      --success: #28a745;
      --warning: #ffc107;
      --danger: #dc3545;
      --card-light: #fff;
      --card-dark: #2c2c2c;
    }
    body {
      margin: 0;
      font-family: 'Cairo', sans-serif;
      background: var(--bg-light);
      color: var(--text-light);
      transition: all 0.3s ease;
    }
    body.dark { background: var(--bg-dark); color: var(--text-dark); }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 25px;
      background: var(--primary);
      color: white;
    }
    .theme-toggle, .export-btn, .import-btn, .save-btn {
      background: var(--secondary);
      border: none;
      padding: 8px 14px;
      border-radius: 20px;
      cursor: pointer;
      color: white;
      font-weight: bold;
      margin-left: 10px;
    }
    .container {
      display: flex;
      gap: 20px;
      padding: 20px;
    }
    .folders, .tasks {
      border-radius: 15px;
      padding: 20px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
      background: var(--card-light);
    }
    body.dark .folders, body.dark .tasks { background: var(--card-dark); }
    .folders { width: 25%; }
    .tasks { flex: 1; }
    .task {
      border-bottom: 1px solid #ccc;
      padding: 10px 0;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .task:last-child { border-bottom: none; }
    .completed { text-decoration: line-through; opacity: 0.6; }
    .task-actions button {
      margin-left: 5px;
      padding: 5px 10px;
      border-radius: 6px;
      border: none;
      cursor: pointer;
      color: white;
    }
    .complete-btn { background: var(--success); }
    .edit-btn { background: var(--warning); }
    .delete-btn { background: var(--danger); }
    .search-bar, .filter-bar, .sort-bar {
      margin: 10px 0;
      display: flex;
      gap: 10px;
    }
    input, select, button {
      padding: 8px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-family: inherit;
    }
    button:hover { opacity: 0.8; }
    .folder-item {
      padding: 10px;
      background: var(--primary);
      color: white;
      margin: 5px 0;
      border-radius: 8px;
      cursor: pointer;
    }
    .folder-item.active { background: var(--secondary); }
  </style>
</head>
<body>
  <header>
    <h1>📋 برنامج المهام المتطور برو</h1>
    <div>
      <button class="theme-toggle">🌙 تبديل الثيم</button>
      <button class="save-btn" onclick="saveData()">💾 حفظ الآن</button>
      <button class="export-btn" onclick="exportData()">💽 تصدير</button>
      <button class="import-btn" onclick="importData()">📂 استيراد</button>
    </div>
  </header>

  <div class="container">
    <!-- المجلدات -->
    <div class="folders">
      <h2>📂 المجلدات</h2>
      <div id="folderList"></div>
      <input type="text" id="folderInput" placeholder="أدخل اسم المجلد">
      <button onclick="addFolder()">➕ إضافة مجلد</button>
    </div>

    <!-- المهام -->
    <div class="tasks">
      <h2>📝 المهام</h2>
      <!-- شريط الأدوات -->
      <div class="search-bar">
        <input type="text" id="searchInput" placeholder="🔍 بحث عن مهمة..." onkeyup="renderTasks()">
      </div>
      <div class="filter-bar">
        <select id="filterSelect" onchange="renderTasks()">
          <option value="all">الكل</option>
          <option value="completed">المكتملة</option>
          <option value="pending">غير المكتملة</option>
        </select>
        <select id="sortSelect" onchange="renderTasks()">
          <option value="date">ترتيب بالتاريخ</option>
          <option value="importance">ترتيب بالأهمية</option>
        </select>
      </div>
      <!-- قائمة المهام -->
      <div id="taskList"></div>

      <!-- إضافة مهمة -->
      <div class="add-task">
        <input type="text" id="taskInput" placeholder="أدخل المهمة">
        <input type="date" id="taskDate">
        <select id="taskImportance">
          <option value="عالية">أهمية عالية</option>
          <option value="متوسطة">أهمية متوسطة</option>
          <option value="منخفضة">أهمية منخفضة</option>
        </select>
        <select id="taskDifficulty">
          <option value="سهل">سهل</option>
          <option value="متوسط">متوسط</option>
          <option value="صعب">صعب</option>
        </select>
        <button onclick="addTask()">➕ إضافة مهمة</button>
      </div>
    </div>
  </div>

  <script>
    let folders = JSON.parse(localStorage.getItem("folders")) || ["عام","الدراسة","العمل"];
    let tasks = JSON.parse(localStorage.getItem("tasks")) || {};
    let currentFolder = folders[0];

    const folderList = document.getElementById("folderList");
    const taskList = document.getElementById("taskList");

    function saveData() {
      localStorage.setItem("folders", JSON.stringify(folders));
      localStorage.setItem("tasks", JSON.stringify(tasks));
      alert("✅ تم الحفظ بنجاح!");
    }

    function renderFolders() {
      folderList.innerHTML = "";
      folders.forEach(folder => {
        const div = document.createElement("div");
        div.className = "folder-item" + (folder === currentFolder ? " active" : "");
        div.textContent = folder;
        div.onclick = () => { currentFolder = folder; renderFolders(); renderTasks(); };
        folderList.appendChild(div);
      });
    }

    function renderTasks() {
      taskList.innerHTML = "";
      if (!tasks[currentFolder]) tasks[currentFolder] = [];
      let list = [...tasks[currentFolder]];

      const search = document.getElementById("searchInput").value.toLowerCase();
      list = list.filter(t => t.text.toLowerCase().includes(search));

      const filter = document.getElementById("filterSelect").value;
      if (filter === "completed") list = list.filter(t => t.completed);
      if (filter === "pending") list = list.filter(t => !t.completed);

      const sort = document.getElementById("sortSelect").value;
      if (sort === "date") list.sort((a,b) => new Date(a.date||0) - new Date(b.date||0));
      if (sort === "importance") {
        const order = {"عالية":1,"متوسطة":2,"منخفضة":3};
        list.sort((a,b) => order[a.importance]-order[b.importance]);
      }

      list.forEach((task,i) => {
        const div = document.createElement("div");
        div.className = "task" + (task.completed ? " completed" : "");
        div.innerHTML = `
          <span><strong>${task.text}</strong> <br>
          ⏰ ${task.date || "بدون موعد"} | ⭐️ ${task.importance} | 🎯 ${task.difficulty}</span>
          <div class="task-actions">
            <button class="complete-btn" onclick="toggleComplete(${i})">${task.completed ? "↩️ استرجاع" : "✅ إكمال"}</button>
            <button class="edit-btn" onclick="editTask(${i})">✏️ تعديل</button>
            <button class="delete-btn" onclick="deleteTask(${i})">🗑 حذف</button>
          </div>
        `;
        taskList.appendChild(div);
      });
    }

    function addFolder() {
      const folderName = document.getElementById("folderInput").value.trim();
      if (!folderName) return alert("أدخل اسم المجلد");
      if (!folders.includes(folderName)) {
        folders.push(folderName);
        tasks[folderName] = [];
      }
      document.getElementById("folderInput").value = "";
      renderFolders(); saveData();
    }

    function addTask() {
      const text = document.getElementById("taskInput").value;
      const date = document.getElementById("taskDate").value;
      const importance = document.getElementById("taskImportance").value;
      const difficulty = document.getElementById("taskDifficulty").value;
      if (!text.trim()) return alert("الرجاء إدخال مهمة");
      tasks[currentFolder].push({text,date,importance,difficulty,completed:false});
      document.getElementById("taskInput").value = "";
      document.getElementById("taskDate").value = "";
      renderTasks();
    }

    function toggleComplete(i) { tasks[currentFolder][i].completed = !tasks[currentFolder][i].completed; renderTasks(); }
    function deleteTask(i) { tasks[currentFolder].splice(i,1); renderTasks(); }
    function editTask(i) {
      const newText = prompt("تعديل المهمة:", tasks[currentFolder][i].text);
      if (newText) tasks[currentFolder][i].text = newText;
      renderTasks();
    }

    function exportData() {
      const data = JSON.stringify({folders,tasks});
      const blob = new Blob([data], {type:"application/json"});
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "tasks_backup.json";
      link.click();
    }

    function importData() {
      const input = document.createElement("input");
      input.type = "file";
      input.accept = ".json";
      input.onchange = e => {
        const file = e.target.files[0];
        const reader = new FileReader();
        reader.onload = () => {
          const data = JSON.parse(reader.result);
          folders = data.folders; tasks = data.tasks;
          currentFolder = folders[0];
          renderFolders(); renderTasks();
        };
        reader.readAsText(file);
      };
      input.click();
    }

    const themeToggle = document.querySelector(".theme-toggle");
    themeToggle.addEventListener("click", () => {
      document.body.classList.toggle("dark");
      localStorage.setItem("theme", document.body.classList.contains("dark") ? "dark":"light");
    });
    if (localStorage.getItem("theme") === "dark") document.body.classList.add("dark");

    renderFolders(); renderTasks();
  </script>
</body>
</html>
