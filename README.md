
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Control de Ingresos y Gastos</title>
  <style>
    body { font-family: Arial, sans-serif; background:#f6f6f6; margin:0; padding:20px; }
    .container { max-width:900px; margin:auto; background:#fff; padding:20px; border-radius:12px; box-shadow:0 0 10px #0002; }
    h1 { font-size:24px; }
    input, select, button { padding:10px; border-radius:6px; border:1px solid #ccc; }
    .row { display:flex; gap:10px; flex-wrap:wrap; margin-bottom:15px; }
    .box { flex:1; min-width:250px; }
    .list { max-height:250px; overflow:auto; background:#fafafa; padding:10px; border-radius:8px; border:1px solid #ddd; }
    .item { display:flex; justify-content:space-between; margin-bottom:8px; padding:6px; background:#fff; border-radius:6px; border:1px solid #eee; }
    .totales { text-align:right; font-weight:bold; margin-top:10px; }
    .btn-whatsapp { background:#25D366; color:#fff; border:none; }
    .btn-danger { background:#ffdddd; color:#c00; border:1px solid #c77; }
    .btn-primary { background:#0066ff; color:#fff; border:none; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Control Mensual de Ingresos y Gastos</h1>
    <div id="month"></div>
    <h2 id="balance"></h2>

    <div class="row">
      <div class="box">
        <label>Concepto</label>
        <input id="label" class="w-full" placeholder="Ej: Salario o Luz" />
      </div>

      <div class="box">
        <label>Monto</label>
        <input id="amount" type="number" step="0.01" placeholder="0.00" />
      </div>

      <div class="box">
        <label>Tipo</label>
        <select id="type">
          <option value="income">Ingreso</option>
          <option value="expense">Gasto</option>
        </select>
      </div>
    </div>

    <button class="btn-primary" onclick="addItem()">Agregar</button>

    <hr><br>

    <div class="row">
      <div class="box">
        <h3>Ingresos</h3>
        <div id="incomeList" class="list"></div>
        <div id="totalIncome" class="totales"></div>
      </div>
      <div class="box">
        <h3>Gastos</h3>
        <div id="expenseList" class="list"></div>
        <div id="totalExpense" class="totales"></div>
      </div>
    </div>

    <br>
    <div class="row">
      <button class="btn-whatsapp" onclick="shareWhatsApp()">Compartir por WhatsApp</button>
      <button onclick="exportCSV()">Exportar CSV</button>
      <button class="btn-danger" onclick="clearAll()">Borrar Todo</button>
    </div>
  </div>

  <script>
    let incomeItems = JSON.parse(localStorage.getItem("incomeItems")) || [];
    let expenseItems = JSON.parse(localStorage.getItem("expenseItems")) || [];

    const monthName = new Date().toLocaleString('es', { month:'long', year:'numeric' });
    document.getElementById("month").innerHTML = `<strong>${monthName}</strong>`;

    function save() {
      localStorage.setItem("incomeItems", JSON.stringify(incomeItems));
      localStorage.setItem("expenseItems", JSON.stringify(expenseItems));
    }

    function format(n){ return Number(n).toFixed(2); }

    function addItem(){
      const label = document.getElementById("label").value.trim();
      const amount = parseFloat(document.getElementById("amount").value);
      const type = document.getElementById("type").value;

      if(!label || !amount || amount <= 0) return;

      const item = { id: Date.now(), label, amount };
      if(type === "income") incomeItems.push(item);
      else expenseItems.push(item);

      save();
      render();

      document.getElementById("label").value = "";
      document.getElementById("amount").value = "";
    }

    function removeItem(id, type){
      if(type === 'income') incomeItems = incomeItems.filter(i => i.id !== id);
      else expenseItems = expenseItems.filter(i => i.id !== id);
      save();
      render();
    }

    function render(){
      const incomeList = document.getElementById("incomeList");
      const expenseList = document.getElementById("expenseList");

      incomeList.innerHTML = incomeItems.map(it => `
        <div class='item'>
          <div>${it.label} - ${format(it.amount)}</div>
          <button onclick="removeItem(${it.id}, 'income')">X</button>
        </div>`).join('');

      expenseList.innerHTML = expenseItems.map(it => `
        <div class='item'>
          <div>${it.label} - ${format(it.amount)}</div>
          <button onclick="removeItem(${it.id}, 'expense')">X</button>
        </div>`).join('');

      const totalIn = incomeItems.reduce((s,i)=>s+i.amount,0);
      const totalEx = expenseItems.reduce((s,i)=>s+i.amount,0);
      const balance = totalIn - totalEx;

      document.getElementById("totalIncome").innerHTML = `Total: ${format(totalIn)}`;
      document.getElementById("totalExpense").innerHTML = `Total: ${format(totalEx)}`;
      document.getElementById("balance").innerHTML = `Restante: ${format(balance)}`;
    }

    function shareWhatsApp(){
      const totalIn = incomeItems.reduce((s,i)=>s+i.amount,0);
      const totalEx = expenseItems.reduce((s,i)=>s+i.amount,0);
      const balance = totalIn - totalEx;

      const text = encodeURIComponent(
        `Resumen ${monthName}\n\nIngresos: ${format(totalIn)}\nGastos: ${format(totalEx)}\nRestante: ${format(balance)}`
      );

      window.open(`https://api.whatsapp.com/send?text=${text}`,"_blank");
    }

    function exportCSV(){
      let rows = "tipo,concepto,monto\n";

      incomeItems.forEach(i => rows += `Ingreso,${i.label},${i.amount}\n`);
      expenseItems.forEach(i => rows += `Gasto,${i.label},${i.amount}\n`);

      const blob = new Blob([rows], { type:"text/csv" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "finanzas.csv";
      a.click();
      URL.revokeObjectURL(url);
    }

    function clearAll(){
      if(!confirm("¿Seguro que quieres borrar todo?")) return;
      incomeItems = [];
      expenseItems = [];
      save();
      render();
    }

    render();
  </script>
</body>
</html>

