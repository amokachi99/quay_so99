<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Vòng quay 10 đối tượng (Loại trừ trùng)</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      margin: 0; padding: 0;
      background: #f5f5f5;
    }
    h1 {
      margin-top: 1rem;
    }
    #inputArea {
      margin: 1rem auto;
      width: 300px;
    }
    textarea {
      width: 100%;
      height: 120px;
      resize: none;
    }
    #wheelContainer {
      margin: 1rem auto;
      position: relative;
      width: 300px;
      height: 300px;
      border: 5px solid #ccc;
      border-radius: 50%;
      overflow: hidden;
    }
    /* Bản thân bánh xe */
    #wheel {
      width: 100%; height: 100%;
      position: absolute; top: 0; left: 0;
      transform: rotate(0deg);
      transition: transform 6s ease-out; /* Sẽ thay đổi bằng JS */
    }
    .wheel-item {
      position: absolute;
      width: 60px; height: 60px;
      line-height: 60px;
      border-radius: 50%;
      background: #ffcc66;
      color: #333;
      font-weight: bold;
      left: 50%; top: 50%;
      transform-origin: -30px -30px;
      text-align: center;
      border: 2px solid #999;
    }

    /* Mũi tên cố định */
    #pointer {
      position: absolute;
      width: 0; height: 0;
      border-left: 15px solid transparent;
      border-right: 15px solid transparent;
      border-bottom: 20px solid red; 
      left: 50%; top: -20px;
      transform: translateX(-50%);
    }

    #spinBtn {
      display: none; 
      margin: 1rem;
      padding: 0.5rem 1rem;
      font-size: 1rem;
    }

    /* Kết quả hiển thị */
    #resultContainer {
      margin: 1rem;
      display: none;
    }
    #resultText {
      font-size: 1.5rem;
      font-weight: bold;
      margin-bottom: 1rem;
    }

    /* Pháo hoa đơn giản: nhấp nháy nhiều màu xung quanh text */
    @keyframes fireworks {
      0%   { text-shadow: 0 0 5px red;    color: red; }
      25%  { text-shadow: 0 0 5px green;  color: green; }
      50%  { text-shadow: 0 0 5px blue;   color: blue; }
      75%  { text-shadow: 0 0 5px orange; color: orange; }
      100% { text-shadow: 0 0 5px red;    color: red; }
    }
    .fireworks {
      animation: fireworks 1s infinite;
    }

    /* Danh sách trúng thưởng */
    #winnersLog {
      width: 60%;
      margin: 1rem auto 2rem auto;
      text-align: left;
      background: #fff;
      padding: 1rem;
      box-shadow: 0 0 5px rgba(0,0,0,0.1);
    }
    #winnersLog h2 {
      margin-top: 0;
    }
    .winner-entry {
      margin: 0.5rem 0;
      font-size: 1rem;
    }
    .timestamp {
      color: #888;
      font-size: 0.9rem;
    }
  </style>
</head>
<body>

<h1>Danh sách - Vòng Quay Nhận Hội:</h1>
<div id="inputArea">
  <textarea id="nameList" placeholder="Mỗi dòng 1 tên..."></textarea><br>
  <button onclick="setupWheel()">Xác nhận danh sách</button>
</div>

<div id="wheelContainer">
  <div id="pointer"></div>
  <div id="wheel"></div>
</div>

<button id="spinBtn" onclick="spin()">Quay</button>

<div id="resultContainer">
  <div id="resultText"></div>
  <button onclick="resetWheel()">Quay lại</button>
</div>

<!-- Danh sách trúng thưởng -->
<div id="winnersLog" style="display:none;">
  <h2>Kết quả quay (theo thứ tự)</h2>
  <div id="logContent"></div>
</div>

<script>
  // Mảng các đối tượng (items) còn lại để quay
  let items = [];

  // Góc hiện tại của bánh xe
  let currentAngle = 0;

  // setupWheel() đọc dữ liệu từ textarea, hiển thị bánh xe
  function setupWheel() {
    const text = document.getElementById("nameList").value.trim();
    if (!text) {
      alert("Hãy nhập ít nhất 1 tên!");
      return;
    }
    // Tách dòng
    let rawItems = text.split("\n").map(s => s.trim()).filter(s => s);
    // Chỉ lấy tối đa 10
    rawItems = rawItems.slice(0,10);

    if (rawItems.length < 2) {
      alert("Cần ít nhất 2 đối tượng để quay!");
      return;
    }

    // Gán vào mảng items
    items = rawItems;
    // Vẽ lại bánh xe
    drawWheel(items);

    // Hiển thị nút Quay, ẩn kết quả
    document.getElementById("spinBtn").style.display = "inline-block";
    document.getElementById("resultContainer").style.display = "none";

    // Hiển thị phần log nếu có
    const wl = document.getElementById("winnersLog");
    wl.style.display = "block"; // Cho chắc chắn, để luôn xem lịch sử
  }

  // drawWheel: vẽ các item xung quanh bánh xe
  function drawWheel(arr) {
    const wheel = document.getElementById("wheel");
    wheel.innerHTML = ""; // xóa cũ
    currentAngle = 0;
    wheel.style.transform = `rotate(${currentAngle}deg)`;
    wheel.style.transition = "";

    const N = arr.length;
    for (let i=0; i<N; i++) {
      const angle = (360 / N) * i;
      const div = document.createElement("div");
      div.className = "wheel-item";
      div.textContent = arr[i];
      // Đặt transform
      div.style.transform = `rotate(${angle}deg) translate(115px) rotate(-${angle}deg)`;
      wheel.appendChild(div);
    }
  }

  // spin() => quay trong khoảng [5..10] giây, chọn 1 đối tượng ngẫu nhiên
  function spin() {
    if (items.length === 0) {
      alert("Tất cả đã được trúng thưởng, không còn đối tượng để quay!");
      return;
    }

    const N = items.length;
    if (N === 1) {
      // Chỉ còn 1 đối tượng => auto trúng luôn
      showResult(0, true); 
      return;
    }

    // 1) Chọn thời gian quay [5..10]
    const duration = Math.random() * 8 + 8; // (8..16)
    
    // 2) Chọn index ngẫu nhiên
    const randomIndex = Math.floor(Math.random() * N);

    // 3) Tính góc để item randomIndex dừng ở vị trí 0°
    const extraRounds = Math.floor(Math.random()*3) + 4; // 4..6 vòng
    const itemAngle = (360/N)*randomIndex;
    let targetAngle = 360*extraRounds - itemAngle;

    // Quay từ currentAngle -> finalAngle
    const finalAngle = currentAngle + targetAngle;

    const wheel = document.getElementById("wheel");
    wheel.style.transition = `transform ${duration}s ease-out`;
    wheel.style.transform = `rotate(${finalAngle}deg)`;

    // Bắt sự kiện khi quay xong
    wheel.ontransitionend = () => {
      wheel.ontransitionend = null;
      // cập nhật currentAngle
      currentAngle = finalAngle % 360;
      // show kết quả
      showResult(randomIndex, false);
    };
  }

  // showResult => gỡ item ra khỏi items, hiển thị
  function showResult(index, autoSingle) {
    // autoSingle: nếu = true, nghĩa là còn đúng 1 item
    let chosen;
    if (autoSingle) {
      // Còn 1 item => index = 0
      chosen = items[0];
      items.splice(0,1); 
    } else {
      chosen = items[index];
      items.splice(index,1);
    }

    // Ẩn nút Quay nếu items đã hết
    if (items.length === 0) {
      document.getElementById("spinBtn").style.display = "none";
      alert("Đã quay hết! Không còn đối tượng để quay.");
    }

    // Vẽ lại bánh xe với items còn lại
    drawWheel(items);

    // Hiển thị kết quả
    document.getElementById("resultContainer").style.display = "block";
    const rt = document.getElementById("resultText");
    rt.textContent = "Kết quả: " + chosen;
    rt.classList.add("fireworks");

    // Ghi log
    addWinnerLog(chosen);
  }

  // resetWheel: ẩn kết quả, cho phép quay tiếp (nếu còn item)
  function resetWheel() {
    document.getElementById("resultContainer").style.display = "none";
    document.getElementById("resultText").classList.remove("fireworks");

    // Nếu vẫn còn items thì hiện nút quay
    if (items.length > 0) {
      document.getElementById("spinBtn").style.display = "inline-block";
    }
  }

  // Ghi vào danh sách trúng thưởng, kèm ngày giờ
  function addWinnerLog(name) {
    // Lấy thời điểm hiện tại
    const now = new Date();
    const timestr = now.toLocaleString(); // "21/04/2023, 14:12:45" tùy trình duyệt

    const logContent = document.getElementById("logContent");

    // Tạo 1 div chứa thông tin
    const div = document.createElement("div");
    div.className = "winner-entry";
    div.innerHTML = `<span class="timestamp">[${timestr}]</span> - <strong>${name}</strong>`;

    logContent.appendChild(div);
  }
</script>

</body>
</html>
