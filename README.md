# SNACK
<!doctype html>
<html lang="th">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Late Night Snack – Order</title>
<style>
  :root{--brand:#6a3f2b;}
  body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial; background:#fff7ef; margin:0; color:#2b2b2b;}
  header{padding:18px 14px; background:#ffe9cf; border-bottom:3px solid var(--brand)}
  h1{margin:0; font-size:1.4rem; color:var(--brand)}
  .wrap{max-width:900px;margin:auto;padding:14px}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:12px}
  .card{background:#fff;border:1.5px solid #ead8c8;border-radius:12px;padding:12px}
  .row{display:flex;justify-content:space-between;gap:8px;align-items:center}
  .name{font-weight:600}
  .price{color:#a3602e}
  input[type=number]{width:84px;padding:6px 8px;border:1px solid #d8c6b8;border-radius:8px}
  input[type=text]{width:100%;padding:10px;border:1px solid #d8c6b8;border-radius:10px}
  .totals{margin-top:14px;padding:12px;border-radius:12px;background:#fff;border:2px dashed #e7cdb4}
  .totals strong{font-size:1.15rem}
  button{cursor:pointer;background:var(--brand);color:#fff;border:none;padding:12px 16px;border-radius:12px;font-weight:700}
  .actions{display:flex;gap:10px;flex-wrap:wrap}
  small{color:#6b6b6b}
  .muted{color:#7a6a63}
</style>
</head>
<body>
<header>
  <h1>Late Night Snack – สั่งออนไลน์</h1>
  <div class="wrap muted">หอลำดวน 4 ถึงหน้าห้อง • 20:00–ดึก • ชำระปลายทาง / โอน</div>
</header>

<div class="wrap">
  <!-- Customer -->
  <div class="card" style="margin-bottom:12px">
    <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px">
      <label>ชื่อผู้สั่ง
        <input type="text" id="custName" placeholder="เช่น แจ็ค">
      </label>
      <label>ห้อง
        <input type="text" id="room" placeholder="เช่น C-402">
      </label>
    </div>
    <div style="margin-top:8px">
      <label>หมายเหตุ (ถ้ามี)
        <input type="text" id="note" placeholder="เช่น ไม่เอาถุง / ให้โทรก่อนถึง">
      </label>
    </div>
  </div>

  <!-- Products -->
  <div class="grid" id="itemsGrid"></div>

  <!-- Totals -->
  <div class="totals" id="summaryBox">
    <div class="row">
      <div><strong>ราคารวม:</strong></div>
      <div><strong id="grand">0</strong> บาท</div>
    </div>
    <small id="smallSummary">ยังไม่ได้เลือกสินค้า</small>
  </div>

  <!-- Actions -->
  <div class="actions" style="margin-top:14px">
    <button id="sendLine">ส่งสรุปไป LINE</button>
    <button id="sendWA" style="background:#25D366">ส่งไป WhatsApp</button>
    <button id="copyText" style="background:#444">คัดลอกสรุป</button>
  </div>
  <small class="muted">* ไปที่ปุ่ม “ส่ง” จะเปิดแอปด้วยข้อความอัตโนมัติ (แก้เบอร์/ไอดีได้ในโค้ด)</small>
</div>

<script>
/** ตั้งค่าร้านค้า **/
const LINE_ID = "YOUR_LINE_ID_OR_LINK"; // ถ้าใช้ line.me/R/ti/p/~<id> ก็ใส่ลิงก์เต็มได้
const WHATSAPP_NUMBER = "66900000000";  // ใส่เบอร์แบบรหัสประเทศ เช่น 6695xxxxxxx

/** เมนูและราคา **/
const PRODUCTS = [
  { id:"mama", name:"มาม่าต้มยำกุ้ง", price:7 },
  { id:"karamucho", name:"คารามูโจ้", price:25 },
  { id:"alps", name:"แอลพี ช็อกโกแลต แท่ง", price:5 },
  { id:"pocky", name:"Pocky ซองเล็ก", price:5 },
  { id:"bigbag", name:"Big Bag สไปซี่", price:20 },
  { id:"squid18", name:"หมึกกรอบ size 18g (ใหญ่)", price:10 },
  { id:"dragonCola", name:"ลิ้นมังกร โคล่า", price:5 },
  { id:"dragonStraw", name:"ลิ้นมังกร สตรอว์เบอร์รี่", price:5 },
  { id:"voiz", name:"Voiz", price:5 },
  { id:"laysbbq", name:"เลย์ รสบาร์บีคิว", price:20 },
];

const grid = document.getElementById("itemsGrid");
PRODUCTS.forEach(p=>{
  const el=document.createElement("div");
  el.className="card";
  el.innerHTML=`
    <div class="row"><div class="name">${p.name}</div><div class="price">${p.price} บาท</div></div>
    <div class="row" style="margin-top:8px">
      <label for="q_${p.id}">จำนวน</label>
      <input type="number" min="0" step="1" value="0" id="q_${p.id}" data-id="${p.id}">
    </div>`;
  grid.appendChild(el);
});

function getOrder(){
  const lines=[];
  let total=0;
  PRODUCTS.forEach(p=>{
    const q = parseInt(document.getElementById("q_"+p.id).value || "0",10);
    if(q>0){
      const line=`- ${p.name} x ${q} = ${p.price*q}฿`;
      lines.push(line);
      total += p.price*q;
    }
  });
  return {lines,total};
}

function renderSummary(){
  const {lines,total} = getOrder();
  document.getElementById("grand").textContent = total.toLocaleString("th-TH");
  const sm = document.getElementById("smallSummary");
  sm.textContent = lines.length ? `รายการ: ${lines.length} • รวม ${total} บาท` : "ยังไม่ได้เลือกสินค้า";
}
grid.addEventListener("input", renderSummary);
renderSummary();

function buildMessage(){
  const name = (document.getElementById("custName").value || "").trim();
  const room = (document.getElementById("room").value || "").trim();
  const note = (document.getElementById("note").value || "").trim();
  const {lines,total} = getOrder();

  if(!name || !room) {
    alert("กรุณากรอกชื่อและเลขห้อง");
    return null;
  }
  if(lines.length===0){
    alert("กรุณาเลือกรายการอย่างน้อย 1 ชิ้น");
    return null;
  }

  const header = `คำสั่งซื้อ Late Night Snack\nชื่อ: ${name}\nห้อง: ${room}`;
  const body = lines.join("\n");
  const footer = `รวมทั้งหมด: ${total} บาท` + (note? `\nหมายเหตุ: ${note}`:"");
  return `${header}\n------------------\n${body}\n------------------\n${footer}`;
}

/** ส่งไป LINE **/
document.getElementById("sendLine").addEventListener("click", ()=>{
  const msg = buildMessage(); if(!msg) return;
  // ถ้ามีลิงก์ไลน์อย่างเป็นทางการให้ใส่ไว้ที่ LINE_ID
  // ใช้ line://msg/text/ สำหรับแชร์ข้อความ
  const url = `line://msg/text/${encodeURIComponent(msg)}`;
  window.location.href = url;
  // ถ้าต้องการเด้งไปที่ไอดีร้าน ให้ใช้ลิงก์ line.me ของร้านในปุ่ม/ลิงก์แยก
});

/** ส่งไป WhatsApp **/
document.getElementById("sendWA").addEventListener("click", ()=>{
  const msg = buildMessage(); if(!msg) return;
  const url = `https://wa.me/${WHATSAPP_NUMBER}?text=${encodeURIComponent(msg)}`;
  window.open(url, "_blank");
});

/** คัดลอกสรุป **/
document.getElementById("copyText").addEventListener("click", async ()=>{
  const msg = buildMessage(); if(!msg) return;
  try{
    await navigator.clipboard.writeText(msg);
    alert("คัดลอกสรุปแล้ว นำไปวางในแชตได้เลย!");
  }catch(e){
    alert("คัดลอกไม่สำเร็จ");
  }
});
</script>
</body>
</html>
