<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Fake News Detection App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background:#f4f4f9;
      text-align:center;
      padding:30px;
    }
    .container {
      background:white;
      padding:20px;
      border-radius:10px;
      width:900px;
      margin:auto;
      box-shadow:0 4px 6px rgba(0,0,0,0.2);
    }
    textarea {
      width:100%;
      height:100px;
      margin-bottom:15px;
      padding:10px;
      font-size:16px;
    }
    button {
      background:#007bff;
      color:white;
      padding:8px 16px;
      border:none;
      border-radius:6px;
      cursor:pointer;
      margin:5px;
    }
    button:hover { background:#0056b3; }
    #result { margin-top:20px; font-size:18px; font-weight:bold; }
    .fake { color:red; }
    .real { color:green; }
    .progress {
      width:100%;
      background:#ddd;
      border-radius:8px;
      margin-top:10px;
    }
    .progress-bar {
      height:20px;
      border-radius:8px;
      text-align:center;
      color:white;
      font-size:12px;
    }
    table {
      width:100%;
      border-collapse:collapse;
      margin-top:20px;
    }
    th, td {
      border:1px solid #ccc;
      padding:8px;
      text-align:left;
    }
    th { background:#007bff; color:white; }
    .highlight { background:yellow; font-weight:bold; }
    .action-btn { background:#28a745; padding:4px 8px; margin:2px; }
    .action-btn:hover { background:#218838; }
    .delete-btn { background:#dc3545; }
    .delete-btn:hover { background:#a71d2a; }
  </style>
</head>
<body>
  <div class="container">
    <h1>📰 Fake News Detection App</h1>
    <textarea id="newsInput" placeholder="Enter news text here..."></textarea><br>
    <button onclick="checkNews()">Check News</button>
    <button onclick="clearHistory()">Clear History</button>
    <button onclick="downloadCSV()">Download CSV</button>
    <button onclick="downloadText()">Download TXT</button>

    <div id="result"></div>

    <h2>📑 History</h2>
    <table id="historyTable">
      <thead>
        <tr>
          <th>#</th>
          <th>News</th>
          <th>Result</th>
          <th>Confidence</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <script>
    let history = [];
    let fakeKeywords = ["shocking", "click here", "you won't believe", "fake", "hoax", "breaking"];

    function highlightKeywords(text) {
      fakeKeywords.forEach(word => {
        let regex = new RegExp("(" + word + ")", "gi");
        text = text.replace(regex, `<span class='highlight'>$1</span>`);
      });
      return text;
    }

    function checkNews() {
      let text = document.getElementById("newsInput").value.trim();
      let resultDiv = document.getElementById("result");

      if (text === "") {
        resultDiv.innerHTML = "⚠️ Please enter some text!";
        resultDiv.style.color = "red";
        return;
      }

      let found = fakeKeywords.some(word => text.toLowerCase().includes(word));
      let confidence = Math.floor(Math.random() * 21) + 80; // 80–100%
      let progressColor = found ? "red" : "green";

      if (found) {
        resultDiv.innerHTML = `<span class="fake">🚨 Fake News Detected</span>
          <div class="progress">
            <div class="progress-bar" style="width:${confidence}%; background:${progressColor};">
              ${confidence}%
            </div>
          </div>
          <p><b>News:</b> ${highlightKeywords(text)}</p>`;
        addToHistory(text, "Fake", confidence);
      } else {
        resultDiv.innerHTML = `<span class="real">✅ Real News</span>
          <div class="progress">
            <div class="progress-bar" style="width:${confidence}%; background:${progressColor};">
              ${confidence}%
            </div>
          </div>
          <p><b>News:</b> ${text}</p>`;
        addToHistory(text, "Real", confidence);
      }

      document.getElementById("newsInput").value = "";
    }

    function addToHistory(news, result, confidence) {
      history.push({ news, result, confidence });
      renderHistory();
    }

    function renderHistory() {
      let tbody = document.querySelector("#historyTable tbody");
      tbody.innerHTML = "";
      history.forEach((item, index) => {
        let row = `<tr>
          <td>${index + 1}</td>
          <td>${highlightKeywords(item.news)}</td>
          <td>${item.result}</td>
          <td>${item.confidence}%</td>
          <td>
            <button class="action-btn" onclick="editNews(${index})">Edit</button>
            <button class="delete-btn" onclick="deleteNews(${index})">Delete</button>
          </td>
        </tr>`;
        tbody.innerHTML += row;
      });
    }

    function editNews(index) {
      let newText = prompt("Edit the news text:", history[index].news);
      if (newText !== null && newText.trim() !== "") {
        history[index].news = newText;
        renderHistory();
      }
    }

    function deleteNews(index) {
      history.splice(index, 1);
      renderHistory();
    }

    function clearHistory() {
      history = [];
      renderHistory();
    }

    function downloadCSV() {
      let csv = "No,News,Result,Confidence\n";
      history.forEach((item, index) => {
        csv += `${index + 1},"${item.news.replace(/"/g, '""')}",${item.result},${item.confidence}%\n`;
      });
      let blob = new Blob([csv], { type: "text/csv" });
      let url = window.URL.createObjectURL(blob);
      let a = document.createElement("a");
      a.href = url;
      a.download = "fake_news_history.csv";
      a.click();
    }

    function downloadText() {
      let textData = "Fake News Detection History\n\n";
      history.forEach((item, index) => {
        textData += `${index + 1}. News: ${item.news}\n   Result: ${item.result}\n   Confidence: ${item.confidence}%\n\n`;
      });
      let blob = new Blob([textData], { type: "text/plain" });
      let url = window.URL.createObjectURL(blob);
      let a = document.createElement("a");
      a.href = url;
      a.download = "fake_news_history.txt";
      a.click();
    }
  </script>
</body>
</html>
