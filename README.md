<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>シフト管理システム</title>
  <style>
    body {
      font-family: sans-serif;
      padding: 20px;
    }
    input, button {
      margin: 5px;
      padding: 5px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      border: 1px solid #ccc;
      text-align: center;
      padding: 8px;
    }
    th {
      background-color: #f2f2f2;
    }
    #adminSection {
      display: none;
      margin-top: 40px;
    }
  </style>
</head>
<body>

  <h2>シフト管理システム</h2>

  <label>日付: <input type="date" id="date"></label>
  <label>氏名: <input type="text" id="name"></label><br>

  <button onclick="recordTime('start')">出勤</button>
  <button onclick="recordTime('breakStart')">休憩開始</button>
  <button onclick="recordTime('breakEnd')">休憩終了</button>
  <button onclick="recordTime('end')">退勤</button>

  <table id="shiftTable">
    <thead>
      <tr>
        <th>日付</th>
        <th>氏名</th>
        <th>出勤</th>
        <th>休憩開始</th>
        <th>休憩終了</th>
        <th>退勤</th>
      </tr>
    </thead>
    <tbody>
      <!-- 記録がここに追加されます -->
    </tbody>
  </table>

  <h3>管理画面ログイン</h3>
  <label>パスコード: <input type="password" id="adminCode"></label>
  <button onclick="loginAdmin()">ログイン</button>

  <div id="adminSection">
    <h3>管理画面</h3>
    <table id="adminTable">
      <thead>
        <tr>
          <th>日付</th>
          <th>氏名</th>
          <th>勤務時間（分）</th>
          <th>休憩時間（分）</th>
          <th>給与（円）</th>
        </tr>
      </thead>
      <tbody>
        <!-- 管理データがここに追加されます -->
      </tbody>
    </table>
  </div>

  <script>
    const records = [];

    function recordTime(type) {
      const date = document.getElementById("date").value;
      const name = document.getElementById("name").value.trim();
      if (!date || !name) {
        alert("日付と氏名を入力してください。");
        return;
      }

      let record = records.find(r => r.date === date && r.name === name);
      if (!record) {
        record = { date, name, start: "", breakStart: "", breakEnd: "", end: "" };
        records.push(record);
      }

      const now = new Date();
      const time = now.toTimeString().slice(0,5); // HH:MM
      record[type] = time;

      renderTable();
    }

    function renderTable() {
      const tbody = document.getElementById("shiftTable").getElementsByTagName("tbody")[0];
      tbody.innerHTML = "";

      const sorted = [...records].sort((a, b) => {
        if (a.date === b.date) return a.name.localeCompare(b.name);
        return a.date.localeCompare(b.date);
      });

      for (const record of sorted) {
        const row = tbody.insertRow();
        row.insertCell(0).textContent = record.date;
        row.insertCell(1).textContent = record.name;
        row.insertCell(2).textContent = record.start || "-";
        row.insertCell(3).textContent = record.breakStart || "-";
        row.insertCell(4).textContent = record.breakEnd || "-";
        row.insertCell(5).textContent = record.end || "-";
      }
    }

    function loginAdmin() {
      const code = document.getElementById("adminCode").value;
      if (code === "SHUTL") {
        document.getElementById("adminSection").style.display = "block";
        renderAdminTable();
      } else {
        alert("パスコードが間違っています。");
      }
    }

    function renderAdminTable() {
      const tbody = document.getElementById("adminTable").getElementsByTagName("tbody")[0];
      tbody.innerHTML = "";

      for (const record of records) {
        const start = parseTime(record.start);
        const end = parseTime(record.end);
        const breakStart = parseTime(record.breakStart);
        const breakEnd = parseTime(record.breakEnd);

        let workMinutes = 0;
        let breakMinutes = 0;

        if (start && end) {
          workMinutes = (end - start) / 60000;
        }

        if (breakStart && breakEnd) {
          breakMinutes = (breakEnd - breakStart) / 60000;
        }

        const netWorkMinutes = workMinutes - breakMinutes;
        const salary = Math.floor((netWorkMinutes / 60) * 1200);

        const row = tbody.insertRow();
        row.insertCell(0).textContent = record.date;
        row.insertCell(1).textContent = record.name;
        row.insertCell(2).textContent = netWorkMinutes > 0 ? netWorkMinutes : "-";
        row.insertCell(3).textContent = breakMinutes > 0 ? breakMinutes : "-";
        row.insertCell(4).textContent = salary > 0 ? salary : "-";
      }
    }

    function parseTime(timeStr) {
      if (!timeStr) return null;
      const [hours, minutes] = timeStr.split(":").map(Number);
      const now = new Date();
      now.setHours(hours, minutes, 0, 0);
      return now;
    }
  </script>

</body>
</html>
