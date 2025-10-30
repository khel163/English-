<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>गाँव समिति चंदा और खर्चा रजिस्टर</title>
    <style>
        body { font-family: 'Noto Sans Devanagari', Arial, sans-serif; background-color: #f5f5f5; margin: 0; padding: 0; }
        header { background-color: #007b5e; color: white; text-align: center; padding: 1rem; }
        h2 { color: #007b5e; margin-left: 1rem; }
        .form-section { background-color: white; padding: 1rem; margin: 1rem; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        input, button { margin: 5px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; font-size: 1rem; }
        button { background-color: #007b5e; color: white; cursor: pointer; }
        button:hover { background-color: #005a43; }
        table { width: 95%; margin: 1rem auto; border-collapse: collapse; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 0 5px rgba(0,0,0,0.1);}
        th, td { border: 1px solid #ddd; padding: 10px 15px; text-align: left; }
        th { background-color: #007b5e; color: white; }
        .edit-btn, .delete-btn { margin: 0 5px; padding: 5px 10px; border: none; border-radius: 4px; cursor: pointer; font-size: 0.9rem;}
        .edit-btn { background-color: #007b5e; color: white; }
        .delete-btn { background-color: #d9534f; color: white; }
        .summary { text-align: center; margin: 20px 0; }
        .donation { font-weight: bold; color: #28a745; font-size: 1.3rem; margin-bottom: 10px; }
        .expense { font-weight: bold; color: #d9534f; font-size: 1.3rem; margin-bottom: 10px; }
        .balance { font-weight: bold; color: #007b5e; font-size: 1.3rem; margin-bottom: 10px; }
        .export-buttons { text-align: center; margin-bottom: 1rem; }
        .export-buttons button { margin: 0 10px; padding: 10px 15px; font-size: 1rem; }
    </style>
</head>
<body>
    <header>
        <h1>गाँव समिति | चंदा और खर्चा रजिस्टर</h1>
    </header>

    <section class="form-section">
        <h2>चंदा जोड़ें</h2>
        <form id="donationForm">
            <input type="text" id="name" placeholder="देनदार का नाम" required />
            <input type="number" id="amount" placeholder="राशि (₹)" required />
            <input type="date" id="date" required />
            <button type="submit">जोड़ें</button>
        </form>
    </section>

    <section class="form-section">
        <h2>खर्च जोड़ें</h2>
        <form id="expenseForm">
            <input type="text" id="expenseName" placeholder="खर्च का नाम" required />
            <input type="number" id="expenseAmount" placeholder="राशि (₹)" required />
            <input type="date" id="expenseDate" required />
            <button type="submit">जोड़ें</button>
        </form>
    </section>

    <section>
        <h2>चंदा का विवरण</h2>
        <table id="donationTable">
            <thead>
                <tr>
                    <th>नाम</th>
                    <th>राशि (₹)</th>
                    <th>तारीख</th>
                    <th>क्रियाएं</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </section>

    <section>
        <h2>खर्च का विवरण</h2>
        <table id="expenseTable">
            <thead>
                <tr>
                    <th>नाम</th>
                    <th>राशि (₹)</th>
                    <th>तारीख</th>
                    <th>क्रियाएं</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </section>

    <section class="summary">
        <h2>सारांश</h2>
        <div id="totalDonationDisplay" class="donation"></div>
        <div id="totalExpenseDisplay" class="expense"></div>
        <div id="balanceDisplay" class="balance"></div>
    </section>
    <section class="export-buttons">
        <button id="printButton">प्रिंट करें</button>
        <button id="downloadPdf">डाउनलोड PDF</button>
        <button id="downloadExcel">डाउनलोड Excel</button>
    </section>
    <footer>
        <p>© 2025 गाँव समिति | सभी अधिकार सुरक्षित</p>
    </footer>

    <!-- jsPDF और autoTable plugin -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <!-- SheetJS Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const donationForm = document.getElementById('donationForm');
            const expenseForm = document.getElementById('expenseForm');
            const donationTable = document.querySelector('#donationTable tbody');
            const expenseTable = document.querySelector('#expenseTable tbody');
            const totalDonationDisplay = document.getElementById('totalDonationDisplay');
            const totalExpenseDisplay = document.getElementById('totalExpenseDisplay');
            const balanceDisplay = document.getElementById('balanceDisplay');

            let donations = JSON.parse(localStorage.getItem('donations')) || [];
            let expenses = JSON.parse(localStorage.getItem('expenses')) || [];

            function updateLocalStorage() {
                localStorage.setItem('donations', JSON.stringify(donations));
                localStorage.setItem('expenses', JSON.stringify(expenses));
            }

            function updateSummary() {
                const totalDonation = donations.reduce((sum, d) => sum + parseFloat(d.amount), 0);
                const totalExpense = expenses.reduce((sum, e) => sum + parseFloat(e.amount), 0);
                const balance = totalDonation - totalExpense;

                totalDonationDisplay.textContent = `कुल मिला चंदा: ₹${totalDonation.toFixed(2)}`;
                totalExpenseDisplay.textContent = `कुल खर्च: ₹${totalExpense.toFixed(2)}`;
                balanceDisplay.textContent = `कुल बचा हुआ पैसा: ₹${balance.toFixed(2)}`;
            }

            function renderTables() {
                donationTable.innerHTML = '';
                expenseTable.innerHTML = '';

                donations.forEach((item, i) => {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td>${item.name}</td>
                        <td>₹${parseFloat(item.amount).toFixed(2)}</td>
                        <td>${item.date}</td>
                        <td>
                            <button class="edit-btn" onclick="editItem('donation',${i})">Edit</button>
                            <button class="delete-btn" onclick="deleteItem('donation',${i})">Delete</button>
                        </td>`;
                    donationTable.appendChild(row);
                });

                expenses.forEach((item, i) => {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td>${item.name}</td>
                        <td>₹${parseFloat(item.amount).toFixed(2)}</td>
                        <td>${item.date}</td>
                        <td>
                            <button class="edit-btn" onclick="editItem('expense',${i})">Edit</button>
                            <button class="delete-btn" onclick="deleteItem('expense',${i})">Delete</button>
                        </td>`;
                    expenseTable.appendChild(row);
                });

                updateSummary();
            }

            window.deleteItem = function(type, index) {
                if (confirm('क्या आप वाकई इस रिकॉर्ड को हटाना चाहते हैं?')) {
                    if (type === 'donation') donations.splice(index, 1);
                    else expenses.splice(index, 1);
                    updateLocalStorage();
                    renderTables();
                }
            };

            window.editItem = function(type, index) {
                const data = type === 'donation' ? donations[index] : expenses[index];
                const name = prompt('नाम:', data.name);
                const amount = prompt('राशि:', data.amount);
                const date = prompt('तारीख (YYYY-MM-DD):', data.date);
                if (name && amount && date) {
                    data.name = name;
                    data.amount = amount;
                    data.date = date;
                    updateLocalStorage();
                    renderTables();
                }
            };

            donationForm.addEventListener('submit', e => {
                e.preventDefault();
                const name = document.getElementById('name').value;
                const amount = document.getElementById('amount').value;
                const date = document.getElementById('date').value;
                donations.push({ name, amount, date });
                updateLocalStorage();
                renderTables();
                donationForm.reset();
            });

            expenseForm.addEventListener('submit', e => {
                e.preventDefault();
                const name = document.getElementById('expenseName').value;
                const amount = document.getElementById('expenseAmount').value;
                const date = document.getElementById('expenseDate').value;
                expenses.push({ name, amount, date });
                updateLocalStorage();
                renderTables();
                expenseForm.reset();
            });

            document.getElementById('printButton').addEventListener('click', () => {
                let printContent = `<h2>चंदा का विवरण</h2>` + document.getElementById('donationTable').outerHTML;
                printContent += `<h2>खर्च का विवरण</h2>` + document.getElementById('expenseTable').outerHTML;
                printContent += `<h2>सारांश</h2><div>${totalDonationDisplay.textContent}</div><div>${totalExpenseDisplay.textContent}</div><div>${balanceDisplay.textContent}</div>`;

                const WinPrint = window.open('', '', 'width=900,height=700');
                WinPrint.document.write('<html><head><title>प्रिंट विवरण</title>');
                WinPrint.document.write('<style>table {width: 100%; border-collapse: collapse; border:1px solid #000;} th,td {border:1px solid #000; padding:10px; text-align:left;}</style>');
                WinPrint.document.write('</head><body >');
                WinPrint.document.write(printContent);
                WinPrint.document.write('</body></html>');
                WinPrint.document.close();
                WinPrint.focus();
                WinPrint.print();
                WinPrint.close();
            });

            document.getElementById('downloadPdf').addEventListener('click', () => {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();

                doc.setFont('helvetica', 'normal');
                doc.text("गाँव समिति चंदा और खर्चा रिपोर्ट", 14, 15);

                doc.autoTable({ 
                    startY: 20,
                    styles: { font: "helvetica", fontStyle: "normal" },
                    head: [['नाम', 'राशि (₹)', 'तारीख']],
                    body: donations.map(d => [d.name, parseFloat(d.amount).toFixed(2), d.date]),
                    theme: 'grid',
                    headStyles: {fillColor: [0, 123, 83]}
                });

                let finalY = doc.lastAutoTable.finalY + 10;

                doc.text("खर्च का विवरण", 14, finalY);

                doc.autoTable({
                    startY: finalY + 5,
                    styles: { font: "helvetica", fontStyle: "normal" },
                    head: [['नाम', 'राशि (₹)', 'तारीख']],
                    body: expenses.map(e => [e.name, parseFloat(e.amount).toFixed(2), e.date]),
                    theme: 'grid',
                    headStyles: {fillColor: [217, 83, 79]}
                });

                finalY = doc.lastAutoTable.finalY + 10;
                const totalDonation = donations.reduce((sum, d) => sum + parseFloat(d.amount), 0);
                const totalExpense = expenses.reduce((sum, e) => sum + parseFloat(e.amount), 0);
                const balance = totalDonation - totalExpense;

                doc.text(`कुल मिला चंदा: ₹${totalDonation.toFixed(2)}`, 14, finalY);
                doc.text(`कुल खर्च: ₹${totalExpense.toFixed(2)}`, 14, finalY + 8);
                doc.text(`कुल बचा हुआ पैसा: ₹${balance.toFixed(2)}`, 14, finalY + 16);

                doc.save('gaon-samiti-report.pdf');
            });

            document.getElementById('downloadExcel').addEventListener('click', () => {
                const wb = XLSX.utils.book_new();

                const donationSheetData = [
                    ["नाम", "राशि (₹)", "तारीख"],
                    ...donations.map(d => [d.name, parseFloat(d.amount), d.date])
                ];
                const expenseSheetData = [
                    ["नाम", "राशि (₹)", "तारीख"],
                    ...expenses.map(e => [e.name, parseFloat(e.amount), e.date])
                ];

                const donationSheet = XLSX.utils.aoa_to_sheet(donationSheetData);
                const expenseSheet = XLSX.utils.aoa_to_sheet(expenseSheetData);

                XLSX.utils.book_append_sheet(wb, donationSheet, "चंदा");
                XLSX.utils.book_append_sheet(wb, expenseSheet, "खर्च");

                XLSX.writeFile(wb, "gaon-samiti-report.xlsx");
            });

            renderTables();
        });
    </script>
</body>
</html>
