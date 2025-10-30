<!DOCTYPE html>
<html lang="hi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>गाँव लेखा</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Noto+Sans+Devanagari&display=swap">
  <style>
    body { font-family: 'Noto Sans Devanagari', sans-serif; background: #f5f5f5; margin: 0; padding: 0; }
    header { background: #007b5e; color: white; text-align: center; padding: 1rem; }
    h2 { color: #007b5e; margin-left: 1rem; }
    .form-section { background: white; padding: 1rem; margin: 1rem; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1);}
    input, button { margin: 5px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; font-size: 1rem; }
    button { background: #007b5e; color: white; cursor: pointer; }
    button:hover { background: #005a43; }
    table { width: 95%; margin: 1rem auto; border-collapse: collapse; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 0 5px rgba(0,0,0,0.1);}
    th, td { border: 1px solid #ddd; padding: 10px 15px; text-align: left; }
    th { background: #007b5e; color: white; }
    .edit-btn,.delete-btn,.save-btn,.cancel-btn { margin: 0 5px; padding: 5px 10px; border: none; border-radius: 4px; cursor: pointer; font-size: 0.9rem;}
    .edit-btn { background: #007b5e; color: white; }
    .delete-btn { background: #d9534f; color: white; }
    .save-btn { background: #5cb85c; color: white; }
    .cancel-btn { background: #f0ad4e; color: white; }
    .summary { text-align: center; margin: 20px 0; }
    .donation { font-weight: bold; color: #28a745; font-size: 1.3rem; margin-bottom: 10px;}
    .expense { font-weight: bold; color: #d9534f; font-size: 1.3rem; margin-bottom: 10px;}
    .balance { font-weight: bold; color: #007b5e; font-size: 1.3rem; margin-bottom: 10px;}
    .exportbtns{ text-align:center;}
    .exportbtns button{margin-left:10px;}
  </style>
  <!-- PDF & Excel Export Libraries -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
</head>
<body>
  <header>
    <h1>ग्राम धनपुरी चंदा रजिस्टर</h1>
  </header>

  <section class="form-section">
    <h2>आमदनी जोड़ें</h2>
    <form id="donationForm">
      <input type="text" id="name" placeholder="नाम" required>
      <input type="number" id="amount" placeholder="राशि" required>
      <input type="date" id="date" required>
      <button type="submit">जोड़ें</button>
    </form>
  </section>

  <section class="form-section">
    <h2>खर्च जोड़ें</h2>
    <form id="expenseForm">
      <input type="text" id="expenseName" placeholder="खर्च का नाम" required>
      <input type="number" id="expenseAmount" placeholder="राशि" required>
      <input type="date" id="expenseDate" required>
      <button type="submit">जोड़ें</button>
    </form>
  </section>

  <section>
    <div class="exportbtns">
      <button id="exportPDFButton">PDF डाउनलोड करें</button>
      <button id="exportExcelButton">Excel डाउनलोड करें</button>
      <button id="printButton">प्रिंट करें</button>
    </div>
  </section>

  <section>
    <h2>आमदनी</h2>
    <table id="donationTable">
      <thead>
        <tr>
          <th>नाम</th>
          <th>राशि</th>
          <th>तारीख</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <section>
    <h2>खर्च</h2>
    <table id="expenseTable">
      <thead>
        <tr>
          <th>नाम</th>
          <th>राशि</th>
          <th>तारीख</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <section class="summary">
    <div id="totalDonationDisplay" class="donation"></div>
    <div id="totalExpenseDisplay" class="expense"></div>
    <div id="balanceDisplay" class="balance"></div>
  </section>

  <footer style="text-align:center; margin-bottom:1rem;">
    <p>2025 &copy; गाँव लेखा, Powered by HTML-JS</p>
  </footer>

  <script>
    //--- UTILITIES FOR LOCALSTORAGE ---
    function saveDataToLocalStorage() {
      const donations = [];
      document.querySelectorAll('#donationTable tbody tr').forEach(row => {
        const cols = row.querySelectorAll('td');
        if(cols.length > 2) {
          donations.push({
            name: cols[0].textContent,
            amount: cols[1].textContent,
            date: cols[2].textContent
          });
        }
      });
      const expenses = [];
      document.querySelectorAll('#expenseTable tbody tr').forEach(row => {
        const cols = row.querySelectorAll('td');
        if(cols.length > 2) {
          expenses.push({
            name: cols[0].textContent,
            amount: cols[1].textContent,
            date: cols[2].textContent
          });
        }
      });
      localStorage.setItem('donations', JSON.stringify(donations));
      localStorage.setItem('expenses', JSON.stringify(expenses));
    }

    function loadDataFromLocalStorage() {
      const donations = JSON.parse(localStorage.getItem('donations') || '[]');
      const expenses = JSON.parse(localStorage.getItem('expenses') || '[]');
      donations.forEach(d => addDonationRecord(d.name, d.amount, d.date, false));
      expenses.forEach(e => addExpenseRecord(e.name, e.amount, e.date, false));
    }

    //--- MAIN APP LOGIC ---
    document.addEventListener('DOMContentLoaded', function () {
      const donationForm = document.getElementById('donationForm');
      const expenseForm = document.getElementById('expenseForm');
      const donationTable = document.querySelector('#donationTable tbody');
      const expenseTable = document.querySelector('#expenseTable tbody');
      const totalDonationDisplay = document.getElementById('totalDonationDisplay');
      const totalExpenseDisplay = document.getElementById('totalExpenseDisplay');
      const balanceDisplay = document.getElementById('balanceDisplay');
      let totalDonation = 0;
      let totalExpense = 0;

      function updateSummary() {
        const balance = totalDonation - totalExpense;
        totalDonationDisplay.textContent = "कुल आमदनी: ₹" + totalDonation.toFixed(2);
        totalExpenseDisplay.textContent = "कुल खर्च: ₹" + totalExpense.toFixed(2);
        balanceDisplay.textContent = "बचत: ₹" + balance.toFixed(2);
      }

      function createActionButton(text, className, onClick) {
        const btn = document.createElement('button');
        btn.textContent = text;
        btn.className = className;
        btn.type = "button";
        btn.addEventListener('click', onClick);
        return btn;
      }

      function addDonationRecord(name, amount, date, saveit=true) {
        const row = document.createElement('tr');
        row.innerHTML = `<td>${name}</td><td>${parseFloat(amount).toFixed(2)}</td><td>${date}</td>`;
        const actionsTd = document.createElement('td');
        const editBtn = createActionButton("Edit", "edit-btn", () => editRecord(row, 'donation'));
        const deleteBtn = createActionButton("Delete", "delete-btn", () => deleteRecord(row, 'donation'));
        actionsTd.appendChild(editBtn);
        actionsTd.appendChild(deleteBtn);
        row.appendChild(actionsTd);
        donationTable.appendChild(row);
        totalDonation += parseFloat(amount);
        updateSummary();
        if(saveit!==false) saveDataToLocalStorage();
      }
      function addExpenseRecord(name, amount, date, saveit=true) {
        const row = document.createElement('tr');
        row.innerHTML = `<td>${name}</td><td>${parseFloat(amount).toFixed(2)}</td><td>${date}</td>`;
        const actionsTd = document.createElement('td');
        const editBtn = createActionButton("Edit", "edit-btn", () => editRecord(row, 'expense'));
        const deleteBtn = createActionButton("Delete", "delete-btn", () => deleteRecord(row, 'expense'));
        actionsTd.appendChild(editBtn);
        actionsTd.appendChild(deleteBtn);
        row.appendChild(actionsTd);
        expenseTable.appendChild(row);
        totalExpense += parseFloat(amount);
        updateSummary();
        if(saveit!==false) saveDataToLocalStorage();
      }

      function editRecord(row, type) {
        const cells = row.querySelectorAll('td');
        cells.forEach((cell, index) => {
          if(index === 3) return;
          const currentValue = cell.textContent;
          const input = document.createElement('input');
          input.type = (index == 2) ? 'date' : (index == 1) ? 'number' : 'text';
          input.value = (index == 1) ? parseFloat(currentValue) : currentValue;
          input.defaultValue = currentValue;
          cell.textContent = '';
          cell.appendChild(input);
        });
        const actionsTd = row.querySelector('td:last-child');
        actionsTd.innerHTML = '';
        const saveBtn = createActionButton("Save", "save-btn", () => saveRecord(row, type));
        const cancelBtn = createActionButton("Cancel", "cancel-btn", () => cancelEdit(row, type));
        actionsTd.appendChild(saveBtn);
        actionsTd.appendChild(cancelBtn);
        // Subtract old amount from total to avoid double counting
        const oldAmountText = cells[1].querySelector('input').defaultValue;
        if(type === 'donation') totalDonation -= parseFloat(oldAmountText);
        else if(type === 'expense') totalExpense -= parseFloat(oldAmountText);
        updateSummary();
      }

      function saveRecord(row, type) {
        const inputs = row.querySelectorAll('input');
        inputs.forEach((input, index) => {
          let value = input.value;
          if(index == 1) value = parseFloat(value).toFixed(2);
          input.parentElement.textContent = value;
        });
        // Restore buttons
        const cells = row.querySelectorAll('td');
        const amount = parseFloat(cells[1].textContent);
        const actionsTd = row.querySelector('td:last-child');
        actionsTd.innerHTML = '';
        const editBtn = createActionButton("Edit", "edit-btn", () => editRecord(row, type));
        const deleteBtn = createActionButton("Delete", "delete-btn", () => deleteRecord(row, type));
        actionsTd.appendChild(editBtn);
        actionsTd.appendChild(deleteBtn);

        if(type === 'donation') totalDonation += amount;
        else if(type === 'expense') totalExpense += amount;
        updateSummary();
        saveDataToLocalStorage();
      }

      function cancelEdit(row, type) {
        const cells = row.querySelectorAll('td');
        cells.forEach((cell, index) => {
          if(index === 3) return;
          const input = cell.querySelector('input');
          if(input) cell.textContent = input.defaultValue;
        });
        // Restore buttons
        const actionsTd = row.querySelector('td:last-child');
        actionsTd.innerHTML = '';
        const editBtn = createActionButton("Edit", "edit-btn", () => editRecord(row, type));
        const deleteBtn = createActionButton("Delete", "delete-btn", () => deleteRecord(row, type));
        actionsTd.appendChild(editBtn);
        actionsTd.appendChild(deleteBtn);

        // Re-add old amount if canceling edit
        const amount = parseFloat(cells[1].textContent);
        if(type === 'donation') totalDonation += amount;
        else if(type === 'expense') totalExpense += amount;
        updateSummary();
        saveDataToLocalStorage();
      }

      function deleteRecord(row, type) {
        if(!confirm("डिलीट करना है?")) return;
        const cells = row.querySelectorAll('td');
        const amount = parseFloat(cells[1].textContent);
        if(type === 'donation') totalDonation -= amount;
        else if(type === 'expense') totalExpense -= amount;
        row.remove();
        updateSummary();
        saveDataToLocalStorage();
      }

      donationForm.addEventListener('submit', function (e) {
        e.preventDefault();
        const name = document.getElementById('name').value;
        const amount = document.getElementById('amount').value;
        const date = document.getElementById('date').value;
        addDonationRecord(name, amount, date);
        donationForm.reset();
      });

      expenseForm.addEventListener('submit', function (e) {
        e.preventDefault();
        const name = document.getElementById('expenseName').value;
        const amount = document.getElementById('expenseAmount').value;
        const date = document.getElementById('expenseDate').value;
        addExpenseRecord(name, amount, date);
        expenseForm.reset();
      });

      document.getElementById('printButton').addEventListener('click', function () {
        let printContent = "<h2>आमदनी</h2>" + document.getElementById('donationTable').outerHTML;
        printContent += "<h2>खर्च</h2>" + document.getElementById('expenseTable').outerHTML;
        printContent += "<h2>Summary</h2><div>कुल आमदनी: "+totalDonation.toFixed(2)+"</div><div>कुल खर्च: "+totalExpense.toFixed(2)+"</div><div>बचत: "+(totalDonation-totalExpense).toFixed(2)+"</div>";
        const WinPrint = window.open('', '', 'width=900,height=700');
        WinPrint.document.write('<html><head><title>Print</title><style>table{width:100%;border-collapse:collapse;border:1px solid #000;} th,td{border:1px solid #000;padding:10px;text-align:left;}</style></head><body>');
        WinPrint.document.write(printContent);
        WinPrint.document.write('</body></html>');
        WinPrint.document.close();
        WinPrint.focus();
        WinPrint.print();
        WinPrint.close();
      });

      document.getElementById('exportPDFButton').addEventListener('click', function () {
        html2canvas(document.body).then(function (canvas) {
          const imgData = canvas.toDataURL('image/png');
          const pdf = new window.jspdf.jsPDF('p', 'mm', 'a4');
          const imgProps= pdf.getImageProperties(imgData);
          const pdfWidth = pdf.internal.pageSize.getWidth();
          const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
          pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
          pdf.save("table-data.pdf");
        });
      });

      document.getElementById('exportExcelButton').addEventListener('click', function () {
        function tableToExcel(tableId, filename) {
          let table = document.getElementById(tableId);
          let html = table.outerHTML.replace(/ /g, '%20');
          let url = 'data:application/vnd.ms-excel,' + html;
          let a = document.createElement('a');
          a.href = url;
          a.download = filename;
          a.click();
        }
        tableToExcel('donationTable', 'donation-table.xls');
        tableToExcel('expenseTable', 'expense-table.xls');
      });

      // शुरुआत में लोकल डाटा लोड करें
      loadDataFromLocalStorage();
      updateSummary();
    });
  </script>
</body>
</html>
