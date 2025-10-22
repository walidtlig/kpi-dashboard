<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KPI Dashboard</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.9.1/dist/chart.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f5f5f5;
            color: #333;
            padding: 20px;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
        }
        
        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 32px;
            color: #2c2c2c;
        }
        
        .filters {
            background: white;
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 30px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .filter-group {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
            align-items: center;
        }
        
        .filter-item {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        
        .filter-item label {
            font-weight: 600;
            font-size: 14px;
            color: #555;
        }
        
        .filter-item input,
        .filter-item select {
            padding: 8px 12px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 14px;
            background: white;
            color: #333;
            min-width: 150px;
        }
        
        .filter-item button {
            padding: 10px 20px;
            background: #333;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 600;
            margin-top: 20px;
        }
        
        .filter-item button:hover {
            background: #555;
        }
        
        .kpi-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .kpi-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .kpi-card h3 {
            font-size: 14px;
            color: #666;
            margin-bottom: 10px;
            text-transform: uppercase;
            font-weight: 600;
        }
        
        .kpi-card .value {
            font-size: 36px;
            font-weight: bold;
            color: #2c2c2c;
        }
        
        .kpi-card .subvalue {
            font-size: 12px;
            color: #888;
            margin-top: 5px;
        }
        
        .charts-section {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .chart-container {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .chart-container h3 {
            margin-bottom: 15px;
            font-size: 16px;
            color: #2c2c2c;
        }
        
        .table-container {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            overflow-x: auto;
        }
        
        .table-container h3 {
            margin-bottom: 15px;
            font-size: 16px;
            color: #2c2c2c;
        }
        
        table {
            width: 100%;
            border-collapse: collapse;
        }
        
        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #e0e0e0;
            font-size: 13px;
        }
        
        th {
            background-color: #f9f9f9;
            font-weight: 600;
            color: #555;
        }
        
        tr:hover {
            background-color: #fafafa;
        }
        
        a {
            color: #333;
            text-decoration: none;
            font-weight: 600;
        }
        
        a:hover {
            text-decoration: underline;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
            font-size: 18px;
            color: #666;
        }
        
        .loading-spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #333;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        canvas {
            max-height: 300px;
        }
        
        .error-message {
            background: #ffebee;
            color: #c62828;
            padding: 15px;
            border-radius: 4px;
            margin-bottom: 20px;
        }

        .status-done {
            color: #2e7d32;
            font-weight: 600;
        }

        .status-review {
            color: #f57c00;
            font-weight: 600;
        }

        .last-updated {
            text-align: center;
            color: #888;
            font-size: 12px;
            margin-top: -20px;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Project KPI Dashboard</h1>
        <div class="last-updated" id="lastUpdated">Loading...</div>
        
        <div class="filters">
            <div class="filter-group">
                <div class="filter-item">
                    <label for="dateFrom">Date From:</label>
                    <input type="date" id="dateFrom">
                </div>
                <div class="filter-item">
                    <label for="dateTo">Date To:</label>
                    <input type="date" id="dateTo">
                </div>
                <div class="filter-item">
                    <label for="assigneeFilter">Assignee:</label>
                    <select id="assigneeFilter">
                        <option value="">All Assignees</option>
                    </select>
                </div>
                <div class="filter-item">
                    <label for="statusFilter">Status:</label>
                    <select id="statusFilter">
                        <option value="">All Status</option>
                    </select>
                </div>
                <div class="filter-item">
                    <button id="refreshBtn">↻ Refresh Data</button>
                </div>
            </div>
        </div>
        
        <div id="loading" class="loading">
            <div class="loading-spinner"></div>
            Loading data from Google Sheets...
        </div>
        
        <div id="error" style="display: none;"></div>
        
        <div id="dashboard" style="display: none;">
            <div class="kpi-cards">
                <div class="kpi-card">
                    <h3>Total Tickets</h3>
                    <div class="value" id="totalTickets">0</div>
                </div>
                <div class="kpi-card">
                    <h3>Done Tickets</h3>
                    <div class="value" id="doneTickets">0</div>
                    <div class="subvalue" id="donePercentage">0%</div>
                </div>
                <div class="kpi-card">
                    <h3>In Review</h3>
                    <div class="value" id="reviewTickets">0</div>
                    <div class="subvalue" id="reviewPercentage">0%</div>
                </div>
                <div class="kpi-card">
                    <h3>Total Deliverables</h3>
                    <div class="value" id="totalDeliverables">0</div>
                </div>
                <div class="kpi-card">
                    <h3>Total SKUs</h3>
                    <div class="value" id="totalSKUs">0</div>
                </div>
                <div class="kpi-card">
                    <h3>Avg Deliverables</h3>
                    <div class="value" id="avgDeliverables">0</div>
                    <div class="subvalue">per ticket</div>
                </div>
            </div>
            
            <div class="charts-section">
                <div class="chart-container">
                    <h3>Tickets by Status</h3>
                    <canvas id="statusChart"></canvas>
                </div>
                <div class="chart-container">
                    <h3>Tickets by Assignee</h3>
                    <canvas id="assigneeChart"></canvas>
                </div>
                <div class="chart-container">
                    <h3>Tickets by Project Type</h3>
                    <canvas id="projectTypeChart"></canvas>
                </div>
                <div class="chart-container">
                    <h3>Completion Trend</h3>
                    <canvas id="trendChart"></canvas>
                </div>
            </div>
            
            <div class="table-container">
                <h3>Tickets Details (<span id="tableCount">0</span> records)</h3>
                <table id="ticketsTable">
                    <thead>
                        <tr>
                            <th>Key</th>
                            <th>Summary</th>
                            <th>Assignee</th>
                            <th>Status</th>
                            <th>Resolution</th>
                            <th>Updated</th>
                            <th>Deliverables</th>
                            <th>SKUs</th>
                            <th>Project Type</th>
                        </tr>
                    </thead>
                    <tbody id="ticketsTableBody">
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        let allData = [];
        let filteredData = [];
        let charts = {};

        // IMPORTANT: Replace this with your published CSV URL
        // Go to: File → Share → Publish to web → Choose CSV → Publish → Copy URL
        const PUBLISHED_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vRDFRD2gj_Ea6JswBh-ezGSDXSJ6n_J3UMPqgFvRZ02av1S7SVmvZUk0S9IDjPCXaoJjQl_sHgoSjcl/pub?output=csv';

        function loadData() {
            document.getElementById('loading').style.display = 'block';
            document.getElementById('dashboard').style.display = 'none';
            document.getElementById('error').style.display = 'none';

            console.log('Loading data from:', PUBLISHED_CSV_URL);

            Papa.parse(PUBLISHED_CSV_URL, {
                download: true,
                header: true,
                skipEmptyLines: 'greedy',
                complete: function(results) {
                    console.log('Data loaded successfully!');
                    console.log('Total rows:', results.data.length);
                    
                    if (results.data.length > 0) {
                        console.log('Sample row:', results.data[0]);
                    }
                    
                    allData = results.data.filter(row => {
                        return row.Key && row.Key.trim() !== '';
                    });
                    
                    console.log('Valid tickets:', allData.length);
                    
                    if (allData.length === 0) {
                        showError('No valid data found. Please check your spreadsheet and make sure it\'s published.');
                        document.getElementById('loading').style.display = 'none';
                        return;
                    }
                    
                    updateLastUpdatedTime();
                    initializeFilters();
                    applyFilters();
                    document.getElementById('loading').style.display = 'none';
                    document.getElementById('dashboard').style.display = 'block';
                },
                error: function(error) {
                    console.error('Error loading data:', error);
                    showError('Failed to load data. Make sure the Google Sheet is published as CSV.');
                    document.getElementById('loading').style.display = 'none';
                }
            });
        }

        function updateLastUpdatedTime() {
            const now = new Date();
            document.getElementById('lastUpdated').textContent = 
                `Last updated: ${now.toLocaleString()}`;
        }

        function showError(message) {
            const errorDiv = document.getElementById('error');
            errorDiv.className = 'error-message';
            errorDiv.innerHTML = `<strong>⚠️ Error:</strong> ${message}`;
            errorDiv.style.display = 'block';
        }

        function initializeFilters() {
            const assignees = [...new Set(allData.map(row => row.Assignee).filter(a => a && a.trim() !== ''))].sort();
            const assigneeSelect = document.getElementById('assigneeFilter');
            assigneeSelect.innerHTML = '<option value="">All Assignees</option>';
            assignees.forEach(assignee => {
                const option = document.createElement('option');
                option.value = assignee;
                option.textContent = assignee;
                assigneeSelect.appendChild(option);
            });

            const statuses = [...new Set(allData.map(row => row.Status).filter(s => s && s.trim() !== ''))].sort();
            const statusSelect = document.getElementById('statusFilter');
            statusSelect.innerHTML = '<option value="">All Status</option>';
            statuses.forEach(status => {
                const option = document.createElement('option');
                option.value = status;
                option.textContent = status;
                statusSelect.appendChild(option);
            });
        }

        function applyFilters() {
            const dateFrom = document.getElementById('dateFrom').value;
            const dateTo = document.getElementById('dateTo').value;
            const assignee = document.getElementById('assigneeFilter').value;
            const status = document.getElementById('statusFilter').value;

            filteredData = allData.filter(row => {
                if (dateFrom && row.Updated) {
                    const rowDate = parseDate(row.Updated);
                    if (rowDate && rowDate < new Date(dateFrom)) return false;
                }
                if (dateTo && row.Updated) {
                    const rowDate = parseDate(row.Updated);
                    if (rowDate && rowDate > new Date(dateTo)) return false;
                }
                if (assignee && row.Assignee !== assignee) return false;
                if (status && row.Status !== status) return false;
                return true;
            });

            updateDashboard();
        }

        function parseDate(dateString) {
            if (!dateString) return null;
            const parts = dateString.split('/');
            if (parts.length === 3) {
                return new Date(parts[2], parts[0] - 1, parts[1]);
            }
            return new Date(dateString);
        }

        function isTicketDone(row) {
            const status = (row.Status || '').toLowerCase().trim();
            const resolution = (row.Resolution || '').toLowerCase().trim();
            return status === 'done' && resolution === 'done' && row.Updated;
        }

        function isTicketInReview(row) {
            const status = (row.Status || '').toLowerCase().trim();
            return status === 'review';
        }

        function updateDashboard() {
            updateKPICards();
            updateCharts();
            updateTable();
        }

        function updateKPICards() {
            const total = filteredData.length;
            const done = filteredData.filter(isTicketDone).length;
            const review = filteredData.filter(isTicketInReview).length;
            
            let totalDeliverables = 0;
            let totalSKUs = 0;
            
            filteredData.forEach(row => {
                totalDeliverables += parseInt(row['Number of deliverables']) || 0;
                totalSKUs += parseInt(row['Number of SKUs']) || 0;
            });

            const avgDeliverables = total > 0 ? (totalDeliverables / total).toFixed(1) : 0;

            document.getElementById('totalTickets').textContent = total.toLocaleString();
            document.getElementById('doneTickets').textContent = done.toLocaleString();
            document.getElementById('donePercentage').textContent = total > 0 ? `${Math.round(done/total*100)}% of total` : '0%';
            document.getElementById('reviewTickets').textContent = review.toLocaleString();
            document.getElementById('reviewPercentage').textContent = total > 0 ? `${Math.round(review/total*100)}% of total` : '0%';
            document.getElementById('totalDeliverables').textContent = totalDeliverables.toLocaleString();
            document.getElementById('totalSKUs').textContent = totalSKUs.toLocaleString();
            document.getElementById('avgDeliverables').textContent = avgDeliverables;
        }

        function updateCharts() {
            updateStatusChart();
            updateAssigneeChart();
            updateProjectTypeChart();
            updateTrendChart();
        }

        function updateStatusChart() {
            const statusCount = {};
            filteredData.forEach(row => {
                const status = row.Status || 'Unknown';
                statusCount[status] = (statusCount[status] || 0) + 1;
            });

            const ctx = document.getElementById('statusChart');
            if (charts.statusChart) charts.statusChart.destroy();
            
            charts.statusChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Object.keys(statusCount),
                    datasets: [{
                        label: 'Tickets',
                        data: Object.values(statusCount),
                        backgroundColor: '#666',
                        borderColor: '#333',
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: { legend: { display: false } },
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }

        function updateAssigneeChart() {
            const assigneeCount = {};
            filteredData.forEach(row => {
                const assignee = row.Assignee || 'Unassigned';
                assigneeCount[assignee] = (assigneeCount[assignee] || 0) + 1;
            });

            const ctx = document.getElementById('assigneeChart');
            if (charts.assigneeChart) charts.assigneeChart.destroy();
            
            const entries = Object.entries(assigneeCount).sort((a, b) => b[1] - a[1]).slice(0, 10);
            
            charts.assigneeChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: entries.map(e => e[0]),
                    datasets: [{
                        data: entries.map(e => e[1]),
                        backgroundColor: ['#2c2c2c', '#3d3d3d', '#4e4e4e', '#5f5f5f', '#707070', '#818181', '#929292', '#a3a3a3', '#b4b4b4', '#c5c5c5'],
                        borderColor: '#fff',
                        borderWidth: 2
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: { legend: { position: 'right', labels: { boxWidth: 12, font: { size: 11 } } } }
                }
            });
        }

        function updateProjectTypeChart() {
            const projectTypeCount = {};
            filteredData.forEach(row => {
                const projectType = row['Pens Project Type'] || 'Unknown';
                projectTypeCount[projectType] = (projectTypeCount[projectType] || 0) + 1;
            });

            const ctx = document.getElementById('projectTypeChart');
            if (charts.projectTypeChart) charts.projectTypeChart.destroy();
            
            charts.projectTypeChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Object.keys(projectTypeCount),
                    datasets: [{
                        label: 'Tickets',
                        data: Object.values(projectTypeCount),
                        backgroundColor: '#555',
                        borderColor: '#333',
                        borderWidth: 1
                    }]
                },
                options: {
                    indexAxis: 'y',
                    responsive: true,
                    maintainAspectRatio: true,
                    plugins: { legend: { display: false } },
                    scales: { x: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }

        function updateTrendChart() {
            const dateCount = {};
            filteredData.forEach(row => {
                if (row.Updated) {
                    const date = row.Updated.split(' ')[0];
                    if (!dateCount[date]) {
                        dateCount[date] = { done: 0, review: 0 };
                    }
                    if (isTicketDone(row)) dateCount[date].done++;
                    if (isTicketInReview(row)) dateCount[date].review++;
                }
            });

            const sortedDates = Object.keys(dateCount).sort((a, b) => {
                return parseDate(a) - parseDate(b);
            }).slice(-30);

            const ctx = document.getElementById('trendChart');
            if (charts.trendChart) charts.trendChart.destroy();
            
            charts.trendChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: sortedDates,
                    datasets: [{
                        label: 'Done',
                        data: sortedDates.map(date => dateCount[date].done),
                        borderColor: '#333',
                        backgroundColor: 'rgba(51, 51, 51, 0.1)',
                        tension: 0.4,
                        fill: true
                    }, {
                        label: 'Review',
                        data: sortedDates.map(date => dateCount[date].review),
                        borderColor: '#777',
                        backgroundColor: 'rgba(119, 119, 119, 0.1)',
                        tension: 0.4,
                        fill: true
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    interaction: { mode: 'index', intersect: false },
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }

        function updateTable() {
            const tbody = document.getElementById('ticketsTableBody');
            tbody.innerHTML = '';
            
            document.getElementById('tableCount').textContent = filteredData.length;

            const fragment = document.createDocumentFragment();
            
            filteredData.forEach(row => {
                const tr = document.createElement('tr');
                const jiraLink = `https://jira.atlassian.com/browse/${row.Key}`;
                const statusClass = isTicketDone(row) ? 'status-done' : (isTicketInReview(row) ? 'status-review' : '');
                
                tr.innerHTML = `
                    <td><a href="${jiraLink}" target="_blank" rel="noopener">${row.Key || ''}</a></td>
                    <td>${row.Summary || ''}</td>
                    <td>${row.Assignee || ''}</td>
                    <td class="${statusClass}">${row.Status || ''}</td>
                    <td>${row.Resolution || ''}</td>
                    <td>${row.Updated || ''}</td>
                    <td>${row['Number of deliverables'] || '0'}</td>
                    <td>${row['Number of SKUs'] || '0'}</td>
                    <td>${row['Pens Project Type'] || ''}</td>
                `;
                fragment.appendChild(tr);
            });
            
            tbody.appendChild(fragment);
        }

        document.getElementById('dateFrom').addEventListener('change', applyFilters);
        document.getElementById('dateTo').addEventListener('change', applyFilters);
        document.getElementById('assigneeFilter').addEventListener('change', applyFilters);
        document.getElementById('statusFilter').addEventListener('change', applyFilters);
        document.getElementById('refreshBtn').addEventListener('click', loadData);

        // Auto-load on page load
        window.addEventListener('load', loadData);

        // Auto-refresh every 5 minutes
        setInterval(loadData, 5 * 60 * 1000);
    </script>
</body>
</html>
