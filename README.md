# Productivity--Dashboard
Daily  productivity  tacks eBay and YouTube task dashboard
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>eBay Dashboard - GrandpaRich446</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; padding: 20px; }
        .container { max-width: 1200px; margin: 0 auto; background: white; border-radius: 20px; box-shadow: 0 20px 40px rgba(0,0,0,0.1); overflow: hidden; }
        .header { background: linear-gradient(135deg, #ff6b6b, #feca57); color: white; padding: 30px; text-align: center; }
        .header h1 { font-size: 2.5em; margin-bottom: 10px; }
        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; padding: 30px; }
        .stat-card { background: white; padding: 25px; border-radius: 15px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); text-align: center; }
        .stat-number { font-size: 3em; font-weight: bold; color: #667eea; margin-bottom: 10px; }
        .progress-bar { background: #e0e0e0; height: 20px; border-radius: 10px; overflow: hidden; margin: 15px 0; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, #4ecdc4, #44a08d); transition: width 0.3s ease; border-radius: 10px; }
        .task-section { padding: 30px; border-top: 1px solid #eee; }
        .task-input { display: flex; gap: 10px; margin-bottom: 20px; }
        .task-input input { flex: 1; padding: 15px; border: 2px solid #e0e0e0; border-radius: 10px; font-size: 16px; }
        .task-input select { padding: 15px; border: 2px solid #e0e0e0; border-radius: 10px; background: white; }
        .add-task { background: #667eea; color: white; border: none; padding: 15px 25px; border-radius: 10px; cursor: pointer; font-size: 16px; font-weight: bold; }
        .task-list { list-style: none; }
        .task-item { display: flex; align-items: center; padding: 15px; margin-bottom: 10px; background: #f8f9ff; border-radius: 10px; border-left: 5px solid #667eea; }
        .task-item.high { border-left-color: #ff6b6b; }
        .task-item.medium { border-left-color: #feca57; }
        .task-item input[type="checkbox"] { margin-right: 15px; transform: scale(1.5); }
        .task-text { flex: 1; font-size: 16px; }
        .delete-task { background: #ff6b6b; color: white; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; margin-left: 10px; }
        .goal-inputs { display: flex; gap: 15px; margin-bottom: 20px; flex-wrap: wrap; }
        .goal-inputs input { padding: 12px; border: 2px solid #e0e0e0; border-radius: 8px; font-size: 16px; }
        @media (max-width: 768px) { .stats-grid { grid-template-columns: 1fr; } .goal-inputs { flex-direction: column; } }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📊 eBay & YouTube Dashboard</h1>
            <p>GrandpaRich446 - Daily Business Tracker</p>
        </div>
        
        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-number" id="todayListings">0</div>
                <div>Listings Today</div>
                <div class="progress-bar"><div class="progress-fill" id="listingsProgress" style="width: 0%"></div></div>
                <input type="number" id="dailyListingGoal" placeholder="Daily goal" style="width: 100px; padding: 8px;">
            </div>
            <div class="stat-card">
                <div class="stat-number" id="todayRevenue">$0</div>
                <div>Revenue Today</div>
                <div class="progress-bar"><div class="progress-fill" id="revenueProgress" style="width: 0%"></div></div>
                <input type="number" id="dailyRevenueGoal" placeholder="Revenue goal" style="width: 100px; padding: 8px;">
            </div>
            <div class="stat-card">
                <div class="stat-number" id="youtubeVideos">0</div>
                <div>YouTube Videos</div>
                <div class="progress-bar"><div class="progress-fill" id="youtubeProgress" style="width: 0%"></div></div>
            </div>
            <div class="stat-card">
                <div class="stat-number" id="tasksCompleted">0</div>
                <div>Tasks Completed</div>
            </div>
        </div>

        <div class="task-section">
            <h2>📝 Daily Tasks</h2>
            <div class="goal-inputs">
                <button onclick="addListing()">+1 Listing</button>
                <button onclick="addRevenue()">+$</button>
                <button onclick="addYouTube()">+1 Video</button>
            </div>
            <div class="task-input">
                <input type="text" id="newTask" placeholder="Add new task (e.g., Source thrift store items)">
                <select id="taskPriority">
                    <option value="low">Low</option>
                    <option value="medium">Medium</option>
                    <option value="high">High</option>
                </select>
                <button class="add-task" onclick="addTask()">Add Task</button>
            </div>
            <ul class="task-list" id="taskList"></ul>
        </div>
    </div>

    <script>
        let data = JSON.parse(localStorage.getItem('ebayDashboard')) || {
            listings: 0, revenue: 0, youtube: 0, tasks: [], goals: { listings: 5, revenue: 200 }
        };

        function saveData() {
            localStorage.setItem('ebayDashboard', JSON.stringify(data));
            updateDisplay();
        }

        function updateDisplay() {
            document.getElementById('todayListings').textContent = data.listings;
            document.getElementById('todayRevenue').textContent = `$${data.revenue}`;
            document.getElementById('youtubeVideos').textContent = data.youtube;
            
            const listingProgress = data.goals.listings > 0 ? (data.listings / data.goals.listings * 100) : 0;
            document.getElementById('listingsProgress').style.width = Math.min(100, listingProgress) + '%';
            
            const revenueProgress = data.goals.revenue > 0 ? (data.revenue / data.goals.revenue * 100) : 0;
            document.getElementById('revenueProgress').style.width = Math.min(100, revenueProgress) + '%';
            
            document.getElementById('youtubeProgress').style.width = data.youtube > 3 ? 100 : (data.youtube / 3 * 100) + '%';
            document.getElementById('tasksCompleted').textContent = data.tasks.filter(t => t.completed).length;
            
            document.getElementById('dailyListingGoal').value = data.goals.listings;
            document.getElementById('dailyRevenueGoal').value = data.goals.revenue;
            
            const taskList = document.getElementById('taskList');
            taskList.innerHTML = '';
            data.tasks.forEach((task, index) => {
                const li = document.createElement('li');
                li.className = `task-item ${task.priority}`;
                li.innerHTML = `
                    <input type="checkbox" ${task.completed ? 'checked' : ''} onchange="toggleTask(${index})">
                    <span class="task-text" style="${task.completed ? 'text-decoration: line-through; opacity: 0.6;' : ''}">${task.text}</span>
                    <button class="delete-task" onclick="deleteTask(${index})">Delete</button>
                `;
                taskList.appendChild(li);
            });
        }

        function addListing() {
            data.listings++;
            saveData();
        }

        function addRevenue() {
            const amount = prompt('Enter revenue amount:') || 0;
            data.revenue += parseFloat(amount);
            saveData();
        }

        function addYouTube() {
            data.youtube++;
            saveData();
        }

        function addTask() {
            const text = document.getElementById('newTask').value.trim();
            const priority = document.getElementById('taskPriority').value;
            if (text) {
                data.tasks.unshift({ text, priority, completed: false });
                document.getElementById('newTask').value = '';
                saveData();
            }
        }

        function toggleTask(index) {
            data.tasks[index].completed = !data.tasks[index].completed;
            saveData();
        }

        function deleteTask(index) {
            data.tasks.splice(index, 1);
            saveData();
        }

        document.getElementById('dailyListingGoal').onchange = function() {
            data.goals.listings = parseInt(this.value) || 5;
            saveData();
        };

        document.getElementById('dailyRevenueGoal').onchange = function() {
            data.goals.revenue = parseInt(this.value) || 200;
            saveData();
        };

        // Reset daily counters at midnight
        const now = new Date();
        const today = now.toDateString();
        const lastReset = localStorage.getItem('lastReset');
        if (lastReset !== today) {
            data.listings = 0;
            data.revenue = 0;
            data.youtube = 0;
            localStorage.setItem('lastReset', today);
            saveData();
        }

        updateDisplay();
    </script>
</body>
</html>
