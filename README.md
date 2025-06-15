
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Day Planner with Alerts</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Poppins', sans-serif;
    }
    body {
      margin: 0;
      padding: 0;
      height: 100vh;
      background: linear-gradient(135deg, #00c9ff, #92fe9d);
      background-size: 400% 400%;
      animation: gradientMove 15s ease infinite;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    @keyframes gradientMove {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
    .card {
      background: rgba(255, 255, 255, 0.2);
      backdrop-filter: blur(20px);
      padding: 30px;
      border-radius: 20px;
      width: 100%;
      max-width: 500px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.1);
      color: #fff;
    }
    h2 {
      text-align: center;
      margin-bottom: 25px;
      color: #fff;
    }
    form {
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
    input, select, button {
      padding: 12px;
      border-radius: 10px;
      border: none;
      font-size: 14px;
    }
    input, select {
      background: rgba(255, 255, 255, 0.2);
      color: #fff;
    }
    input::placeholder {
      color: #eee;
    }
    button {
      background-color: #00d084;
      color: white;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.3s ease;
    }
    button:hover {
      background-color: #00a96e;
    }
    #exportBtn {
      background-color: #007bff;
    }
    .task-list {
      margin-top: 25px;
    }
    .task-item {
      background: rgba(255, 255, 255, 0.2);
      padding: 10px;
      border-radius: 8px;
      margin-bottom: 10px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 14px;
    }
    .task-item button {
      background: #ff4d4d;
      padding: 5px 10px;
      border: none;
      color: white;
      border-radius: 6px;
      cursor: pointer;
    }

    .date-time-row {
      display: flex;
      gap: 10px;
    }
    .date-time-row input {
      flex: 1;
    }

    #toast {
      visibility: hidden;
      min-width: 250px;
      background-color: #333;
      color: #fff;
      text-align: center;
      border-radius: 10px;
      padding: 16px;
      position: fixed;
      z-index: 9999;
      left: 50%;
      bottom: 30px;
      transform: translateX(-50%);
      font-size: 16px;
    }
    #toast.show {
      visibility: visible;
      animation: fadein 0.5s, fadeout 0.5s 2.5s;
    }
    @keyframes fadein {
      from { bottom: 0; opacity: 0; }
      to { bottom: 30px; opacity: 1; }
    }
    @keyframes fadeout {
      from { bottom: 30px; opacity: 1; }
      to { bottom: 0; opacity: 0; }
    }
  </style>
</head>
<body>

  <div class="card">
    <h2>📘 Day Planner</h2>
    <form id="taskForm">
      <input type="text" id="taskText" placeholder="Enter task..." required>

      <div class="date-time-row">
        <input type="date" id="taskDate" required>
        <input type="time" id="taskTime" required>
      </div>

      <select id="taskCategory">
        <option value="Work">Work</option>
        <option value="Personal">Personal</option>
        <option value="Study">Study</option>
      </select>

      <button type="submit">✅ Add Task</button>
      <button type="button" id="exportBtn">📤 Export to Excel</button>
    </form>

    <div class="task-list" id="taskList"></div>
  </div>

  <div id="toast"></div>

  <script>
    const form = document.getElementById("taskForm");
    const taskList = document.getElementById("taskList");
    const taskText = document.getElementById("taskText");
    const taskDate = document.getElementById("taskDate");
    const taskTime = document.getElementById("taskTime");
    const taskCategory = document.getElementById("taskCategory");
    const exportBtn = document.getElementById("exportBtn");
    const toast = document.getElementById("toast");

    let tasks = JSON.parse(localStorage.getItem("dayPlannerTasks")) || [];

    function renderTasks() {
      taskList.innerHTML = "";
      tasks.forEach((task, index) => {
        const div = document.createElement("div");
        div.className = "task-item";
        div.innerHTML = `
          <span><strong>${task.date}</strong> ${task.time} — ${task.text} (${task.category})</span>
          <button onclick="deleteTask(${index})">Delete</button>
        `;
        taskList.appendChild(div);
      });
    }

    function deleteTask(index) {
      tasks.splice(index, 1);
      localStorage.setItem("dayPlannerTasks", JSON.stringify(tasks));
      renderTasks();
      showToast("🗑️ Task deleted");
    }

    function showToast(message) {
      toast.textContent = message;
      toast.className = "show";
      setTimeout(() => {
        toast.className = toast.className.replace("show", "");
      }, 3000);
    }

    function scheduleNotification(task) {
      if (!("Notification" in window) || Notification.permission !== "granted") return;

      const taskDateTime = new Date(`${task.date}T${task.time}`);
      const now = new Date();

      const alerts = [
        { minsBefore: 5, label: "🕔 5 mins before" },
        { minsBefore: 1, label: "⏰ 1 min before" }
      ];

      alerts.forEach(alert => {
        const alertTime = taskDateTime.getTime() - alert.minsBefore * 60 * 1000;
        const delay = alertTime - now.getTime();

        if (delay > 0) {
          setTimeout(() => {
            new Notification("🔔 Upcoming Task", {
              body: `${alert.label}: ${task.text} at ${task.time} (${task.category})`,
              icon: "https://cdn-icons-png.flaticon.com/512/1827/1827370.png"
            });
          }, delay);
        }
      });
    }

    function showDailySummary() {
      const now = new Date();
      const hour = now.getHours();
      const minute = now.getMinutes();
      const today = now.toISOString().split("T")[0];
      if (hour === 8 && minute === 0 && Notification.permission === "granted") {
        const todaysTasks = tasks.filter(task => task.date === today);
        if (todaysTasks.length > 0) {
          const body = todaysTasks.map(t => `• ${t.time} - ${t.text} (${t.category})`).join("\n");
          new Notification("📋 Today's Tasks", {
            body,
            icon: "https://cdn-icons-png.flaticon.com/512/1827/1827370.png"
          });
        }
      }
    }

    form.addEventListener("submit", function (e) {
      e.preventDefault();
      const newTask = {
        text: taskText.value,
        date: taskDate.value,
        time: taskTime.value,
        category: taskCategory.value
      };
      tasks.push(newTask);
      localStorage.setItem("dayPlannerTasks", JSON.stringify(tasks));
      renderTasks();
      scheduleNotification(newTask);
      form.reset();
      showToast("✅ Task added!");
    });

    exportBtn.addEventListener("click", () => {
      if (tasks.length === 0) {
        showToast("⚠️ No tasks to export!");
        return;
      }
      const worksheet = XLSX.utils.json_to_sheet(tasks);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Tasks");
      XLSX.writeFile(workbook, "DayPlanner_Tasks.xlsx");
      showToast("📁 Excel file exported!");
    });

    // Ask permission for local notifications
    if ("Notification" in window && Notification.permission !== "granted") {
      Notification.requestPermission().then(permission => {
        if (permission === "granted") {
          console.log("✅ Notifications enabled.");
        }
      });
    }

    // Check for daily summary every minute
    setInterval(showDailySummary, 60 * 1000);

    renderTasks();
  </script>
</body>
</html>

