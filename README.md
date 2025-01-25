# expense-tracker
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Personal Expense Tracker</title>
    <link rel="stylesheet" href="styles.css">
    
        <style>
        :root {
            --bg-color: #f4f4f9;
            --text-color: #000;
            --container-bg: #fff;
            --day-box-bg: #f9f9f9;
            --border-color: #2a14a9;
        }

        [data-theme="dark"] {
            --bg-color: #121212;
            --text-color: #e0e0e0;
            --container-bg: #1e1e1e;
            --day-box-bg: #292929;
            --border-color: #444;
        }

        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: background-color 0.3s, color 0.3s;
        }

        .container {
            width: 90%;
            margin: 20px auto;
            padding: 20px;
            background: var(--container-bg);
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            transition: background-color 0.3s;
        }

        .input-section, .calendar-section, .charts-section {
            margin-top: 20px;
        }

        #calendar {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 10px;
        }

        .day-box {
            border: 1px solid var(--border-color);
            border-radius: 5px;
            padding: 10px;
            background: var(--day-box-bg);
            transition: background-color 0.3s, border-color 0.3s;
        }

        .chart-container {
            width: 100%;
            height: 300px;
        }

        @media screen and (min-width: 768px) {
            .charts-section {
                display: flex;
                gap: 20px;
            }

            .chart-container {
                width: 48%;
            }
        }

        .chart-full {
            width: 100%;
            height: 500px;
        }

        .dark-mode-toggle {
            display: flex;
            justify-content: flex-end;
            align-items: center;
            padding: 10px;
        }

        .dark-mode-toggle label {
            margin-right: 10px;
            font-size: 1rem;
            cursor: pointer;
        }

        .dark-mode-toggle input {
            transform: scale(1.5);
        }
               .intro{
                   display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            background: linear-gradient(135deg, #007bff, #0056b3);
            color: white;
            animation: fadeIn 1s ease-in-out;
        }

        .intro img {
            width: 200px;
            height: auto;
            margin-bottom: 20px;
            animation: bounce 1.5s infinite;
        }

        @keyframes bounce {
            0%, 100% {
                transform: translateY(0);
            }
            50% {
                transform: translateY(-10px);
            }
        }

        .intro button {
            background: #ff9800;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            font-size: 1.2rem;
            color: white;
            cursor: pointer;
            transition: transform 0.3s ease-in-out;
        }

button:hover {
            background: #0056b3;
            transform: scale(1.05);
}
        .intro button:hover {
            transform: scale(1.1);
        }
    </style>
    
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body data-theme="light">
    <div class="dark-mode-toggle">
        <label for="dark-mode">Dark Mode</label>
        <input type="checkbox" id="dark-mode">
    </div>
    
<div id="intro" class="intro">
        <img src="logo.jpg" alt="Tracker Logo">
        <h1>Welcome to Personal Expense Tracker</h1>
    
        <button id="start-button">Let's Get Started</button>
    </div>
    
    <div class="container" id="expense-tracker" style="display: none;">
        <header>
            <h1>Personal Expense Tracker</h1>
        </header>

        <section class="input-section">
            <label for="total-money">Total Money (₹):</label>
            <input type="number" id="total-money" placeholder="Enter total money"><br><br>

            <label for="expense-category">Category:</label>
            <select id="expense-category">
                <option value="Rent">Rent</option>
                <option value="Groceries">Groceries</option>
                <option value="Fuel">Fuel</option>
                <option value="Subscriptions">Subscriptions</option>
                <option value="Shopping">Shopping</option>
                <option value="Traveling">Traveling</option>
                <option value="Other">Other</option>
            </select><br><br>

            <label for="expense-amount">Amount (₹):</label>
            <input type="number" id="expense-amount" placeholder="Enter expense amount"><br><br>

            <label for="expense-date">Date:</label>
            <input type="date" id="expense-date"><br><br>

            <button id="add-expense">Add Expense</button>
        </section>
<hr>
        <section class="calendar-section">
            <h2>Expense Calendar</h2>
            <div id="calendar"></div>
        </section>

        <section class="charts-section">
            <div id="lineChartContainer" class="chart-container">
                <canvas id="lineChart"></canvas>
            </div>
            <div class="chart-container">
                <canvas id="pieChart"></canvas>
            </div>
        </section>
    </div>

    <script>
         const darkModeToggle = document.getElementById('dark-mode');
const body = document.body;

darkModeToggle.addEventListener('change', () => {
    if (darkModeToggle.checked) {
        body.setAttribute('data-theme', 'dark');
    } else {
        body.setAttribute('data-theme', 'light');
    }
});  

const startButton = document.getElementById("start-button");
const trackerSection = document.getElementById("expense-tracker");
const introSection = document.getElementById("intro");
const totalMoneyInput = document.getElementById("total-money");
const categoryInput = document.getElementById("expense-category");
const amountInput = document.getElementById("expense-amount");
const dateInput = document.getElementById("expense-date");
const addExpenseButton = document.getElementById("add-expense");
const calendarContainer = document.getElementById("calendar");
const lineCtx = document.getElementById("lineChart").getContext("2d");
const pieCtx = document.getElementById("pieChart").getContext("2d");

let totalMoney = 0;
let expenses = [];

startButton.addEventListener("click", () => {
    introSection.style.display = "none"; // Hide intro
    trackerSection.style.display = "block"; // Show tracker section
});

// Initialize Charts
const lineChart = new Chart(lineCtx, {
    type: 'line',
    data: { labels: [], datasets: [{ label: 'Daily Expenses', data: [], borderColor: '#007bff', fill: true }] },
    options: { responsive: true, maintainAspectRatio: false, scales: { x: { title: { display: true, text: 'Date' } }, y: { title: { display: true, text: 'Amount (₹)' } } } }
});

const pieChart = new Chart(pieCtx, {
    type: 'pie',
    data: { labels: [], datasets: [{ data: [], backgroundColor: ['#ff6384', '#36a2eb', '#ffce56', '#4bc0c0', '#9966ff', '#ff9f40'] }] },
    options: { responsive: true, maintainAspectRatio: false }
});

// Add Expense
addExpenseButton.addEventListener("click", () => {
    const category = categoryInput.value;
    const amount = parseFloat(amountInput.value);
    const date = dateInput.value;

    if (!category || isNaN(amount) || !date) {
        alert("Please fill all fields!");
        return;
    }

    expenses.push({ category, amount, date });
    
    updateCalendar();
    updateCharts();
});

function updateCalendar() {
    const grouped = groupExpensesByDate();
    calendarContainer.innerHTML = "";

    Object.keys(grouped).forEach(date => {
        const dayBox = document.createElement("div");
        dayBox.className = "day-box";
        dayBox.innerHTML = `<h3>${date}</h3><ul>${grouped[date].map(e => `<li>${e.category}: ₹${e.amount}</li>`).join("")}</ul>`;
        calendarContainer.appendChild(dayBox);
    });
}

function updateCharts() {
    const grouped = groupExpensesByDate();
    const dailyTotals = Object.keys(grouped).map(date => grouped[date].reduce((sum, e) => sum + e.amount, 0));
    const dates = Object.keys(grouped);

    const categoryTotals = groupExpensesByCategory();

    lineChart.data.labels = dates;
    lineChart.data.datasets[0].data = dailyTotals;
    lineChart.update();

    pieChart.data.labels = Object.keys(categoryTotals);
    pieChart.data.datasets[0].data = Object.values(categoryTotals);
    pieChart.update();
}

function groupExpensesByDate() {
    return expenses.reduce((acc, expense) => {
        const date = expense.date;
        if (!acc[date]) acc[date] = [];
        acc[date].push(expense);
        return acc;
    }, {});
}

function groupExpensesByCategory() {
    return expenses.reduce((acc, expense) => {
        const category = expense.category;
        if (!acc[category]) acc[category] = 0;
        acc[category] += expense.amount;
        return acc;
    }, {});
}
    </script>
</body>
</html>
