# -
هاذا الموقع يساعد المنشاة الصغيرة في عملية الفواتير 
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>فاتورة الشراء</title>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f7faff;
      color: #333;
      line-height: 1.8;
      padding: 2rem;
      display: flex;
      justify-content: center;
      align-items: start;
    }

    .container {
      width: 100%;
      max-width: 800px;
      background: white;
      box-shadow: 0 4px 12px rgba(0, 80, 255, 0.1);
      border-radius: 16px;
      padding: 2rem;
      border-right: 6px solid #0073ff;
    }

    h1 {
      color: #0073ff;
      text-align: center;
      font-size: 2rem;
      margin-bottom: 1.5rem;
    }

    label {
      font-weight: bold;
    }

    input {
      width: 100%;
      padding: 0.5rem;
      margin-bottom: 1rem;
      border: 1px solid #ccc;
      border-radius: 8px;
    }

    .btn {
      background-color: #0073ff;
      color: white;
      border: none;
      padding: 0.7rem 1.5rem;
      border-radius: 8px;
      cursor: pointer;
      margin: 0.3rem;
      transition: background 0.3s;
    }

    .btn:hover {
      background-color: #0073ff;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }

    th, td {
      border: 1px solid #ccc;
      padding: 0.7rem;
      text-align: center;
    }

    th {
      background-color: #0073ff;
      color: white;
    }

    /* منطقة الطباعة */
    #printArea {
      display: none;
    }

    @media print {
      body * {
        visibility: hidden;
      }

      #printArea, #printArea * {
        visibility: visible;
      }

      #printArea {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        padding: 20px;
        display: block;
        direction: rtl;
        font-size: 16px;
      }

      .container {
        display: none;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>فاتورة الشراء</h1>
    <label>اسم الشخص:</label>
    <input type="text" id="name" placeholder="أدخل اسم الشخص" />
    <label>اسم الغرض:</label>
    <input type="text" id="item" placeholder="اسم الغرض" />
    <label>سعر المنتج:</label>
    <input type="number" id="price" placeholder="سعر المنتج" />
    <label>الخصم (رقمي):</label>
    <input type="number" id="discount" placeholder="قيمة الخصم" />
    <label>التاريخ:</label>
    <input type="date" id="date" />
    <label>المجموع:</label>
    <input type="text" id="total" readonly />
    <button class="btn" onclick="calculateTotal()">احسب المجموع</button>
    <button class="btn" onclick="saveInvoice()">حفظ الفاتورة</button>
    <button class="btn" onclick="showInvoices()">عرض الفواتير</button>

    <table id="invoiceTable" style="display: none;">
      <thead>
        <tr>
          <th>رقم الفاتورة</th>
          <th>اسم الشخص</th>
          <th>التاريخ</th>
          <th>اسم الغرض</th>
          <th>الخصم</th>
          <th>السعر</th>
          <th>المجموع</th>
          <th>طباعة</th>
          <th>حذف</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div id="printArea"></div>

  <script>
    let invoiceId = Number(localStorage.getItem("invoiceId") || "1");

    function calculateTotal() {
      const price = parseFloat(document.getElementById("price").value) || 0;
      const discount = parseFloat(document.getElementById("discount").value) || 0;
      const total = price - discount;
      document.getElementById("total").value = `${total} ريال`;
    }

    function saveInvoice() {
      const name = document.getElementById("name").value;
      const item = document.getElementById("item").value;
      const price = document.getElementById("price").value + " ريال";
      const discount = document.getElementById("discount").value + " ريال";
      const total = document.getElementById("total").value;
      const date = document.getElementById("date").value;
      const invoice = { id: invoiceId++, name, item, price, discount, total, date };

      if (!name || name === "undefined") {
        alert("يرجى إدخال اسم صحيح");
        return;
      }

      const invoices = JSON.parse(localStorage.getItem("invoices") || "[]");
      invoices.push(invoice);
      localStorage.setItem("invoices", JSON.stringify(invoices));
      localStorage.setItem("invoiceId", invoice.id);
      alert("تم حفظ الفاتورة بنجاح ✅");
    }

    function showInvoices() {
      const invoices = JSON.parse(localStorage.getItem("invoices") || "[]");
      const table = document.getElementById("invoiceTable");
      const tbody = table.querySelector("tbody");
      tbody.innerHTML = "";

      invoices
        .filter(i => i.name && i.name !== "undefined")
        .forEach(i => {
          const row = `
            <tr>
              <td>${i.id}</td>
              <td>${i.name}</td>
              <td>${i.date}</td>
              <td>${i.item}</td>
              <td>${i.discount}</td>
              <td>${i.price}</td>
              <td>${i.total}</td>
              <td><button onclick="printInvoice(${i.id})">طباعة</button></td>
              <td><button onclick="deleteInvoice(${i.id})">حذف</button></td>
            </tr>`;
          tbody.innerHTML += row;
        });

      table.style.display = invoices.length ? "table" : "none";
    }

    function deleteInvoice(id) {
      let invoices = JSON.parse(localStorage.getItem("invoices") || "[]");
      invoices = invoices.filter(i => i.id !== id);
      localStorage.setItem("invoices", JSON.stringify(invoices));
      showInvoices();
    }

    function printInvoice(id) {
      const invoices = JSON.parse(localStorage.getItem("invoices") || "[]");
      const i = invoices.find(i => i.id === id);
      if (!i) return;

      const content = `
        <h2 style="text-align:center;color:#0073ff;">فاتورة الشراء</h2>
        <table style="width:100%;border-collapse:collapse;border:1px solid #000;font-size:16px;text-align:right;">
          <tr><th style="border:1px solid #000;">رقم الفاتورة</th><td style="border:1px solid #000;">${i.id}</td></tr>
          <tr><th style="border:1px solid #000;">التاريخ</th><td style="border:1px solid #000;">${i.date}</td></tr>
          <tr><th style="border:1px solid #000;">اسم الشخص</th><td style="border:1px solid #000;">${i.name}</td></tr>
          <tr><th style="border:1px solid #000;">الغرض</th><td style="border:1px solid #000;">${i.item}</td></tr>
          <tr><th style="border:1px solid #000;">السعر</th><td style="border:1px solid #000;">${i.price}</td></tr>
          <tr><th style="border:1px solid #000;">الخصم</th><td style="border:1px solid #000;">${i.discount}</td></tr>
          <tr><th style="border:1px solid #000;">المجموع</th><td style="border:1px solid #000;">${i.total}</td></tr>
        </table>
      `;

      const printArea = document.getElementById("printArea");
      printArea.innerHTML = content;

      setTimeout(() => {
        window.print();
        printArea.innerHTML = ''; // إفراغ بعد الطباعة لحماية DOM
      }, 100);
    }
  </script>
</body>
</html>
