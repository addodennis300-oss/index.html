
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tenant Maintenance & Payment Portal</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f1f5f9;
  color: #172033;
}
header {
  background: #123b5d;
  color: white;
  padding: 22px 16px;
  text-align: center;
}
header h1 { font-size: 23px; margin: 0 0 8px; }
header p { margin: 0; color: #dbeafe; }
main { max-width: 850px; margin: auto; padding: 16px; }
.card {
  background: white;
  border-radius: 12px;
  padding: 18px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px #0000000d;
}
h2 { font-size: 19px; margin-top: 0; }
label { display: block; margin: 12px 0 6px; font-weight: bold; }
input, select, button {
  width: 100%;
  padding: 12px;
  border-radius: 7px;
  font-size: 16px;
}
input, select { border: 1px solid #cbd5e1; background: white; }
button {
  border: none;
  background: #087f5b;
  color: white;
  font-weight: bold;
  margin-top: 14px;
  cursor: pointer;
}
button.secondary { background: #123b5d; }
button.danger { background: #b42318; }
.summary {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}
.stat {
  padding: 14px;
  border-radius: 9px;
  background: #e8f3ff;
  overflow-wrap: anywhere;
}
.stat strong { display: block; font-size: 20px; margin-top: 8px; }
.small { font-size: 13px; color: #64748b; line-height: 1.5; }
.notice {
  padding: 12px;
  background: #fff4d6;
  border-radius: 8px;
  font-size: 14px;
  line-height: 1.5;
}
.record {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 12px;
  margin-top: 10px;
  overflow-wrap: anywhere;
}
.record p { margin: 6px 0; }
footer { text-align: center; padding: 20px; color: #64748b; }
@media (max-width: 420px) {
  main { padding: 10px; }
  .card { padding: 14px; }
  header h1 { font-size: 21px; }
}
@media print {
  body * { visibility: hidden; }
  #receipt, #receipt * { visibility: visible; }
  #receipt {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
  }
  .no-print { display: none !important; }
}
</style>
</head>
<body>
<header>
  <h1>Tenant Maintenance & Payment Portal</h1>
  <p>Payment records, maintenance, fuel and repairs</p>
  <p>Ghana | Currency: GH₵</p>
</header>

<main>
  <div class="card">
    <div class="notice">
      <strong>Demo version:</strong> Records are saved only in this
      browser on this device. There are no secure accounts or online
      payments yet. Do not enter real tenant or financial information.
    </div>
    <label for="role">Select demo view</label>
    <select id="role">
      <option value="supervisor">Supervisor</option>
      <option value="tenant">Tenant</option>
    </select>
    <label for="tenantFilter">Tenant name</label>
    <input id="tenantFilter" placeholder="Enter tenant name">
    <p class="small">This selection is not a secure login.</p>
  </div>

  <section class="card">
    <h2>Payment Overview</h2>
    <div class="summary">
      <div class="stat">Total recorded
        <strong id="total">GH₵0.00</strong>
      </div>
      <div class="stat">Outstanding balance
        <strong id="balance">GH₵0.00</strong>
      </div>
    </div>
  </section>

  <section class="card" id="paymentCard">
    <h2>Record a Payment</h2>
    <form id="paymentForm">
      <label for="tenant">Tenant name</label>
      <input id="tenant" required placeholder="e.g. Tenant A">
      <label for="category">Payment category</label>
      <select id="category">
        <option>Maintenance Fee</option>
        <option>Fuel Expense</option>
        <option>Repair</option>
      </select>
      <label for="amount">Amount paid (GH₵)</label>
      <input id="amount" type="number" min="0.01" step="0.01"
        required placeholder="0.00">
      <label for="due">Total amount due for this category (GH₵)</label>
      <input id="due" type="number" min="0" step="0.01"
        required placeholder="e.g. 2500">
      <label for="date">Payment date</label>
      <input id="date" type="date" required>
      <label for="method">Payment method</label>
      <select id="method">
        <option>Cash</option>
        <option>MTN Mobile Money</option>
        <option>Telecel Cash</option>
        <option>Bank Transfer</option>
        <option>Other</option>
      </select>
      <label for="reference">Reference or note (optional)</label>
      <input id="reference" placeholder="Demo reference">
      <button type="submit">Save Demo Payment</button>
    </form>
  </section>

  <section class="card">
    <h2>Payment History</h2>
    <p class="small">Use the tenant name filter above to view matching
      demo records.</p>
    <div id="history">No payments recorded yet.</div>
  </section>

  <section class="card" id="receipt">
    <h2>Payment Receipt</h2>
    <div id="receiptContent">
      <p>No receipt selected. Save a demo payment to create one.</p>
    </div>
    <button class="secondary no-print" id="printReceipt" type="button">
      Print / Save Receipt as PDF
    </button>
  </section>

  <section class="card no-print" id="tools">
    <h2>Demo Data Tools</h2>
    <button class="secondary" id="export" type="button">
      Export Records as CSV
    </button>
    <button class="danger" id="clear" type="button">
      Clear All Demo Records
    </button>
    <p class="small">Exported CSV files may contain financial information.
      Keep them private. Clearing records cannot be undone.</p>
  </section>
</main>

<footer>Tenant Maintenance & Payment Portal</footer>

<script>
const $ = id => document.getElementById(id);
const money = n => "GH₵" + Number(n).toFixed(2);
const key = "tenantPortalDemoPayments";
let payments = [];
let selectedReceipt = null;

try {
  payments = JSON.parse(localStorage.getItem(key) || "[]");
  if (!Array.isArray(payments)) payments = [];
} catch (e) {
  payments = [];
}

$("date").value = new Date().toISOString().slice(0, 10);

$("role").addEventListener("change", () => {
  const supervisor = $("role").value === "supervisor";
  $("paymentCard").hidden = !supervisor;
  $("tools").hidden = !supervisor;
  render();
});

$("tenantFilter").addEventListener("input", render);

function save() {
  localStorage.setItem(key, JSON.stringify(payments));
}

function render() {
  const filter = $("tenantFilter").value.trim().toLowerCase();
  const visible = payments.filter(p =>
    !filter || p.tenant.toLowerCase().includes(filter)
  );

  const total = visible.reduce((s, p) => s + p.amount, 0);
  const outstanding = visible.reduce((s, p) => {
    const previous = visible
      .filter(x => x.tenant === p.tenant &&
        x.category === p.category && x.id <= p.id)
      .reduce((sum, x) => sum + x.amount, 0);
    return s;
  }, 0);

  // Each category's latest entered due amount is used.
  const groups = {};
  visible.forEach(p => {
    const k = p.tenant + "|" + p.category;
    if (!groups[k]) groups[k] = { due: p.due, paid: 0 };
    groups[k].due = p.due;
    groups[k].paid += p.amount;
  });
  const balance = Object.values(groups).reduce(
    (s, g) => s + Math.max(0, g.due - g.paid), 0
  );

  $("total").textContent = money(total);
  $("balance").textContent = money(balance);

  $("history").replaceChildren();
  if (!visible.length) {
    $("history").textContent = "No payments recorded yet.";
  }
  visible.slice().reverse().forEach(p => {
    const item = document.createElement("div");
    item.className = "record";
    const title = document.createElement("strong");
    title.textContent = p.tenant + " — " + p.category;
    const details = document.createElement("p");
    details.textContent = p.date + " | " + p.method;
    const amount = document.createElement("p");
    amount.textContent = "Paid: " + money(p.amount);
    const receipt = document.createElement("button");
    receipt.className = "secondary";
    receipt.textContent = "View Receipt";
    receipt.type = "button";
    receipt.onclick = () => showReceipt(p);
    item.append(title, details, amount, receipt);
    $("history").append(item);
  });
}

function showReceipt(p) {
  selectedReceipt = p;
  $("receiptContent").replaceChildren();
  [
    ["Portal", "Tenant Maintenance & Payment Portal"],
    ["Receipt number", p.id],
    ["Tenant", p.tenant],
    ["Category", p.category],
    ["Amount paid", money(p.amount)],
    ["Payment date", p.date],
    ["Payment method", p.method],
    ["Reference / note", p.reference || "Not provided"]
  ].forEach(([label, value]) => {
    const line = document.createElement("p");
    const strong = document.createElement("strong");
    strong.textContent = label + ": ";
    line.append(strong, document.createTextNode(value));
    $("receiptContent").append(line);
  });
}

$("paymentForm").addEventListener("submit", event => {
  event.preventDefault();
  if ($("role").value !== "supervisor") {
    alert("Switch to Supervisor demo view to record payments.");
    return;
  }

  const tenant = $("tenant").value.trim();
  const amount = Number($("amount").value);
  const due = Number($("due").value);

  if (!tenant || amount <= 0 || due < 0) {
    alert("Please check the tenant name and amounts.");
    return;
  }

  const payment = {
    id: "DEMO-" + Date.now(),
    tenant,
    category: $("category").value,
    amount,
    due,
    date: $("date").value,
    method: $("method").value,
    reference: $("reference").value.trim()
  };

  payments.push(payment);
  try {
    save();
  } catch (e) {
    payments.pop();
    alert("Could not save. Browser storage may be unavailable.");
    return;
  }

  showReceipt(payment);
  render();
  $("paymentForm").reset();
  $("date").value = new Date().toISOString().slice(0, 10);
  alert("Demo payment saved on this device.");
});

$("printReceipt").addEventListener("click", () => {
  if (!selectedReceipt) {
    alert("Save a demo payment and select its receipt first.");
    return;
  }
  window.print();
});

$("export").addEventListener("click", () => {
  const fields = ["id", "tenant", "category", "amount",
    "due", "date", "method", "reference"];
  const csvCell = value =>
    '"' + String(value ?? "").replace(/"/g, '""') + '"';
  const csv = [fields.join(","), ...payments.map(p =>
    fields.map(f => csvCell(p[f])).join(",")
  )].join("\r\n");
  const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "demo-payment-records.csv";
  a.click();
  URL.revokeObjectURL(url);
});

$("clear").addEventListener("click", () => {
  if (confirm("Delete every demo payment saved in this browser?")) {
    payments = [];
    selectedReceipt = null;
    save();
    $("receiptContent").textContent = "No receipt selected.";
    render();
  }
});

render();
</script>
</body>
</html>

