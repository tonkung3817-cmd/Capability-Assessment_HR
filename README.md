<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HR Store Capability Assessment 2026</title>
  
  <!-- CSS Frameworks & Libraries -->
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
  
  <!-- TensorFlow.js สำหรับ AI Proctoring -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
  
  <style>
    body { background-color: #f3f4f6; font-family: 'Sarabun', sans-serif; }
    .makro-red { background-color: #990000; }
    .makro-text { color: #990000; }
    .blur-screen { filter: blur(15px); pointer-events: none; user-select: none; }
    #webcam-container { position: fixed; top: 10px; right: 10px; z-index: 50; border: 2px solid #10B981; border-radius: 8px; overflow: hidden; width: 150px; }
    #video { width: 100%; height: auto; }
  </style>
</head>
<body class="antialiased text-gray-800" id="main-body">

  <!-- AI Proctoring Camera -->
  <div id="webcam-container" class="hidden shadow-lg bg-black">
    <video id="video" autoplay muted playsinline></video>
    <div id="ai-status" class="bg-green-500 text-white text-xs text-center font-bold py-1">AI: ปกติ</div>
  </div>

  <nav class="makro-red text-white p-4 shadow-md flex justify-between items-center">
    <h1 class="text-xl font-bold">HR Store Capability Assessment 2026</h1>
    <div id="user-info" class="text-sm font-semibold">Loading...</div>
  </nav>

  <div class="container mx-auto p-4 md:p-6" id="app-content">
    
    <!-- Loading Screen -->
    <div id="loading" class="text-center mt-20">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-red-800 mx-auto"></div>
      <p class="mt-4 text-gray-600 font-semibold">กำลังตรวจสอบสิทธิ์และโหลดข้อมูล...</p>
    </div>

    <!-- Employee / Assessment View -->
    <div id="employee-view" class="hidden">
      <!-- Policy Card -->
      <div class="bg-white p-6 rounded-lg shadow-md mb-6 border-t-4 border-red-800">
        <h2 class="text-lg font-bold text-gray-800 mb-2">คำชี้แจงการประเมิน</h2>
        <p class="text-sm text-gray-600 mb-4">
          แบบประเมินนี้ไม่ได้มุ่งวัดการจดจำหลักการ แต่ให้ความสำคัญกับประสบการณ์ วิธีดำเนินการ และผลลัพธ์จากการทำงานจริง กรุณาตอบคำถามตามหลัก <strong>STAR</strong> (Situation, Task, Action, Result) พร้อมระบุตัวเลขหรือหลักฐานประกอบเท่าที่สามารถเปิดเผยได้
        </p>
        <label class="flex items-start space-x-3 cursor-pointer bg-red-50 p-4 rounded border border-red-200">
          <input type="checkbox" id="accept-policy" class="form-checkbox text-red-600 mt-1 h-5 w-5" onchange="startAssessment()">
          <span class="text-sm font-semibold text-red-800">
            ข้าพเจ้ายอมรับเงื่อนไข เข้าใจว่าการประเมินนี้มีผลต่อการพัฒนาศักยภาพ และอนุญาตให้ระบบเปิดกล้องเว็บแคมเพื่อใช้ AI ป้องกันการทุจริตตลอดการทำแบบประเมิน
          </span>
        </label>
      </div>

      <!-- Questions Container -->
      <div id="questions-container" class="hidden space-y-6"></div>
      
      <!-- Action Buttons -->
      <div id="action-buttons" class="hidden mt-8 flex flex-col md:flex-row justify-between items-center bg-white p-4 rounded-lg shadow">
        <button onclick="saveAssessmentForm(false)" class="w-full md:w-auto bg-gray-500 text-white px-6 py-3 rounded shadow hover:bg-gray-600 font-semibold mb-4 md:mb-0 transition">
          💾 บันทึกร่าง (Save Draft)
        </button>
        <button onclick="saveAssessmentForm(true)" class="w-full md:w-auto bg-red-800 text-white px-8 py-3 rounded shadow hover:bg-red-900 font-bold text-lg transition">
          🚀 ส่งแบบประเมิน (Submit)
        </button>
      </div>
    </div>
  </div>

  <script>
    // --- State Variables ---
    let userContext = {};
    let questionsData = [];
    let isMonitoring = false;
    let currentAssessmentId = '';

    // --- Mock Proxy สำหรับใช้งานในโหมด Preview ของ Editor ---
    const MOCK_PROXY = {
      get run() {
        const proxy = {
          withSuccessHandler: function(cb) { this.successCb = cb; return this; },
          withFailureHandler: function(cb) { this.errorCb = cb; return this; },
          getUserContext: function() {
            setTimeout(() => this.successCb({ email: 'preview@makro.co.th', name: 'คุณประมวล (Preview)', role: 'Administrator' }), 500);
            return this;
          },
          getQuestions: function() {
            const mockQs = [
              { id: 'Q01', dimension: 'Leadership, Capability & Advisory', text: 'ขอให้เล่าตัวอย่างเหตุการณ์ที่คุณพบว่าหัวหน้างานหรือทีมงานในสาขามีช่องว่างด้านทักษะ คุณวิเคราะห์ปัญหา ให้คำแนะนำและติดตามผลอย่างไร?', type: 'Capability' },
              { id: 'Q02', dimension: 'Agility & Execution', text: 'ขอให้เล่าเหตุการณ์ที่คุณต้องรับผิดชอบงาน HR สำคัญหลายเรื่องพร้อมกัน คุณจัดลำดับความสำคัญและประสานงานอย่างไร?', type: 'Capability' },
              { id: 'Q03', dimension: 'Data, Digital & Problem Solving', text: 'ขอให้เล่าตัวอย่างปัญหาด้านบุคลากรที่คุณนำข้อมูลหรือเครื่องมือดิจิทัลมาใช้วิเคราะห์ อธิบายข้อมูลที่ใช้และผลลัพธ์', type: 'Capability' }
            ];
            setTimeout(() => this.successCb(mockQs), 500);
            return this;
          },
          saveAssessment: function(payload, isSubmit) {
            setTimeout(() => this.successCb({ success: true, assessmentId: 'MOCK-ID-1234', status: isSubmit ? 'Submitted' : 'Draft' }), 1000);
            return this;
          }
        };
        return proxy;
      }
    };
    
    // กำหนด Backend API Object (สลับอัตโนมัติ)
    const backendAPI = (typeof google !== 'undefined' && google.script && google.script.run) ? google.script.run : MOCK_PROXY.run;

    // 1. ตรวจสอบข้อมูลเมื่อโหลดหน้าเว็บ
    window.onload = () => {
      backendAPI.withSuccessHandler(initApp).getUserContext();
    };

    function initApp(user) {
      userContext = user;
      document.getElementById('loading').classList.add('hidden');
      
      if(user.role === 'None') {
        Swal.fire({ icon: 'error', title: 'ไม่มีสิทธิ์เข้าถึง', text: 'อีเมลของคุณไม่มีในระบบผู้ทำแบบประเมิน' });
        return;
      }
      
      document.getElementById('user-info').innerText = `${user.name} | ${user.role}`;
      
      // ให้สิทธิ์ HR Store และ Administrator (เพื่อเทส) มองเห็นข้อสอบ
      if(user.role === 'HR Store' || user.role === 'Administrator') {
        document.getElementById('employee-view').classList.remove('hidden');
        backendAPI.withSuccessHandler(renderQuestions).getQuestions();
      }
    }

    // 2. สร้างหน้าต่างข้อสอบจาก Google Sheets
    function renderQuestions(qs) {
      questionsData = qs;
      const container = document.getElementById('questions-container');
      let html = '';
      
      qs.forEach((q, idx) => {
        html += `
          <div class="bg-white p-6 rounded-lg shadow-md border border-gray-200">
            <div class="flex justify-between items-center mb-2">
              <h3 class="font-bold text-lg makro-text">ข้อ ${idx+1}: ${q.dimension}</h3>
              <span class="text-xs bg-gray-200 text-gray-700 px-2 py-1 rounded">${q.type}</span>
            </div>
            <p class="text-sm bg-gray-50 p-4 rounded mb-4 border-l-4 border-red-800 text-gray-700 font-medium">${q.text}</p>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <!-- Situation -->
              <div>
                <label class="text-xs font-bold text-gray-600 mb-1 flex justify-between">
                  <span>S: สถานการณ์ (Situation)</span>
                  <span id="char-S-${q.id}" class="text-gray-400 font-normal">0/80</span>
                </label>
                <textarea id="ans-S-${q.id}" oninput="updateCharCount('${q.id}', 'S', 80)" class="w-full border rounded p-2 text-sm h-28 focus:border-red-500 focus:ring-1 focus:ring-red-500 outline-none" placeholder="อธิบายสถานการณ์หรือปัญหาที่เกิดขึ้น (ขั้นต่ำ 80 ตัวอักษร)"></textarea>
              </div>
              
              <!-- Task -->
              <div>
                <label class="text-xs font-bold text-gray-600 mb-1 flex justify-between">
                  <span>T: บทบาทและความรับผิดชอบ (Task)</span>
                  <span id="char-T-${q.id}" class="text-gray-400 font-normal">0/50</span>
                </label>
                <textarea id="ans-T-${q.id}" oninput="updateCharCount('${q.id}', 'T', 50)" class="w-full border rounded p-2 text-sm h-28 focus:border-red-500 focus:ring-1 focus:ring-red-500 outline-none" placeholder="บทบาทและหน้าที่ของคุณในเหตุการณ์นั้น (ขั้นต่ำ 50 ตัวอักษร)"></textarea>
              </div>

              <!-- Action -->
              <div>
                <label class="text-xs font-bold text-gray-600 mb-1 flex justify-between">
                  <span>A: การลงมือทำ (Action)</span>
                  <span id="char-A-${q.id}" class="text-gray-400 font-normal">0/120</span>
                </label>
                <textarea id="ans-A-${q.id}" oninput="updateCharCount('${q.id}', 'A', 120)" class="w-full border rounded p-2 text-sm h-32 focus:border-red-500 focus:ring-1 focus:ring-red-500 outline-none" placeholder="อธิบายวิธีคิด วิธีแก้ปัญหา และสิ่งที่คุณทำอย่างละเอียด (ขั้นต่ำ 120 ตัวอักษร)"></textarea>
              </div>

              <!-- Result & Evidence -->
              <div class="flex flex-col space-y-2">
                <div>
                  <label class="text-xs font-bold text-gray-600 mb-1 flex justify-between">
                    <span>R: ผลลัพธ์ (Result)</span>
                    <span id="char-R-${q.id}" class="text-gray-400 font-normal">0/80</span>
                  </label>
                  <textarea id="ans-R-${q.id}" oninput="updateCharCount('${q.id}', 'R', 80)" class="w-full border rounded p-2 text-sm h-16 focus:border-red-500 focus:ring-1 focus:ring-red-500 outline-none" placeholder="ผลลัพธ์ที่เกิดจากการกระทำของคุณ (ขั้นต่ำ 80 ตัวอักษร)"></textarea>
                </div>
                <div>
                  <label class="text-xs font-bold text-gray-600 mb-1">E: หลักฐาน (Evidence - ตัวเลือกเสริม)</label>
                  <input type="text" id="ans-E-${q.id}" class="w-full border rounded p-2 text-sm focus:border-red-500 focus:ring-1 focus:ring-red-500 outline-none" placeholder="ระบุตัวเลข % หรืออ้างอิงเอกสารที่ตรวจสอบได้">
                </div>
              </div>
            </div>
          </div>
        `;
      });
      container.innerHTML = html;
    }

    function updateCharCount(qId, part, minReq) {
      const el = document.getElementById(`ans-${part}-${qId}`);
      const counter = document.getElementById(`char-${part}-${qId}`);
      const len = el.value.length;
      counter.innerText = `${len}/${minReq}`;
      counter.className = len >= minReq ? "text-green-600 font-bold" : "text-red-500 font-bold";
    }

    // 3. เริ่ม Assessment และเปิดกล้อง AI Proctoring (พร้อม Bypass สำหรับแอดมิน)
    async function startAssessment() {
      if(!document.getElementById('accept-policy').checked) return;
      document.getElementById('questions-container').classList.remove('hidden');
      document.getElementById('action-buttons').classList.remove('hidden');
      document.getElementById('accept-policy').disabled = true;

      // เริ่ม AI
      try {
        const video = document.getElementById('video');
        // อัปเดต: บังคับการค้นหากล้องหน้าและลดความละเอียดลงเพื่อหลีกเลี่ยงข้อจำกัดของ iFrame
        const stream = await navigator.mediaDevices.getUserMedia({ 
          video: { facingMode: "user", width: { ideal: 320 }, height: { ideal: 240 } } 
        });
        
        video.srcObject = stream;
        document.getElementById('webcam-container').classList.remove('hidden');
        
        Swal.fire({ title: 'กำลังโหลดระบบ AI...', text: 'กรุณารอสักครู่เพื่อเตรียมระบบป้องกันการทุจริต', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
        const model = await cocoSsd.load();
        Swal.close();
        
        isMonitoring = true;
        detectCheating(video, model);
      } catch (err) {
        let errorMessage = 'เบราว์เซอร์ปฏิเสธการเข้าถึงกล้อง';
        let fixInstruction = 'กรุณากดไอคอนรูปแม่กุญแจที่แถบ URL ด้านบนมุมซ้าย จากนั้นเปิดสวิตช์ "กล้องถ่ายรูป (Camera)" เป็นอนุญาต แล้วรีเฟรชหน้าเว็บครับ';
        
        if (err.name === 'NotFoundError' || err.name === 'DevicesNotFoundError') {
          errorMessage = 'ไม่พบกล้องเว็บแคมในอุปกรณ์นี้';
          fixInstruction = 'กรุณาตรวจสอบการเชื่อมต่อกล้อง';
        }

        Swal.fire({
          icon: 'error', 
          title: 'ถูกระบบความปลอดภัยบล็อกกล้อง!', 
          html: `
            <p class="text-red-600 font-bold mb-2">${errorMessage} (${err.name})</p>
            <p class="text-sm text-gray-700 text-left mb-4 bg-gray-100 p-3 rounded border">
              <b>สาเหตุ:</b> Google Apps Script ฝังหน้าเว็บนี้ไว้ใน iFrame ทำให้ Chrome/Edge บล็อกกล้องอัตโนมัติ<br><br>
              <b>วิธีแก้:</b> ${fixInstruction}
            </p>
          `,
          footer: '<button onclick="forceBypassCamera()" class="px-4 py-2 bg-gray-200 text-gray-800 font-semibold rounded shadow hover:bg-gray-300 transition">คลิกที่นี่เพื่อข้ามกล้อง (ทดสอบระบบไปก่อน)</button>'
        });
        
        // คืนค่าปุ่มติ๊กยอมรับ
        document.getElementById('questions-container').classList.add('hidden');
        document.getElementById('action-buttons').classList.add('hidden');
        document.getElementById('accept-policy').checked = false;
        document.getElementById('accept-policy').disabled = false;
      }
    }

    // ฟังก์ชันพิเศษสำหรับแอดมินทดสอบระบบโดยไม่ต้องมีกล้อง
    function forceBypassCamera() {
      Swal.close();
      document.getElementById('questions-container').classList.remove('hidden');
      document.getElementById('action-buttons').classList.remove('hidden');
      document.getElementById('accept-policy').checked = true;
      document.getElementById('accept-policy').disabled = true;
      
      const Toast = Swal.mixin({ toast: true, position: 'top-end', showConfirmButton: false, timer: 3000 });
      Toast.fire({ icon: 'info', title: 'เปิดโหมดทดสอบ (ปิดกล้อง)' });
    }

    // 4. ฟังก์ชัน AI ตรวจจับโทรศัพท์มือถือ (COCO-SSD)
    async function detectCheating(video, model) {
      if (!isMonitoring) return;
      const predictions = await model.detect(video);
      let foundPhone = false;

      predictions.forEach(p => {
        if (p.class === 'cell phone' && p.score > 0.5) foundPhone = true;
      });

      const statusBox = document.getElementById('ai-status');
      if (foundPhone) {
        statusBox.className = 'bg-red-600 text-white text-xs text-center font-bold py-1';
        statusBox.innerText = '⚠️ พบอุปกรณ์สื่อสาร!';
        document.getElementById('main-body').classList.add('blur-screen', 'bg-red-900');
        Swal.fire({
          icon: 'error', title: 'ตรวจพบพฤติกรรมน่าสงสัย!', text: 'ระบบตรวจพบการใช้อุปกรณ์สื่อสาร หน้าจอถูกล็อกชั่วคราว',
          confirmButtonText: 'รับทราบและเก็บอุปกรณ์',
          preConfirm: () => { document.getElementById('main-body').classList.remove('blur-screen', 'bg-red-900'); }
        });
      } else {
        statusBox.className = 'bg-green-500 text-white text-xs text-center font-bold py-1';
        statusBox.innerText = 'AI: ปกติ';
      }
      setTimeout(() => detectCheating(video, model), 2000); // ตรวจทุก 2 วินาที
    }

    // 5. บันทึก / ส่งข้อมูล
    function saveAssessmentForm(isSubmit) {
      // ดึงข้อมูลคำตอบ
      let answers = [];
      let isCompleted = true;

      questionsData.forEach(q => {
        const s = document.getElementById(`ans-S-${q.id}`).value;
        const t = document.getElementById(`ans-T-${q.id}`).value;
        const a = document.getElementById(`ans-A-${q.id}`).value;
        const r = document.getElementById(`ans-R-${q.id}`).value;
        const e = document.getElementById(`ans-E-${q.id}`).value;

        // Validation แบบง่าย (ตรวจสอบความยาว) เฉพาะตอนกด Submit
        if(isSubmit) {
          if (s.length < 80 || t.length < 50 || a.length < 120 || r.length < 80) {
            isCompleted = false;
          }
        }

        answers.push({ questionId: q.id, dimension: q.dimension, situation: s, task: t, action: a, result: r, evidence: e });
      });

      if (isSubmit && !isCompleted) {
        Swal.fire('ข้อมูลไม่ครบถ้วน', 'กรุณาตอบคำถามให้ครบความยาวขั้นต่ำที่กำหนดในทุกช่องของทุกข้อก่อนทำการส่ง', 'warning');
        return;
      }

      Swal.fire({
        title: isSubmit ? 'ยืนยันการส่งแบบประเมิน?' : 'กำลังบันทึกร่าง...',
        text: isSubmit ? 'เมื่อส่งแล้วจะไม่สามารถแก้ไขได้จนกว่าหัวหน้าจะตีกลับ' : 'สามารถกลับมาแก้ไขต่อได้ภายหลัง',
        icon: isSubmit ? 'warning' : 'info',
        showCancelButton: isSubmit,
        confirmButtonText: isSubmit ? 'ยืนยันการส่ง' : 'ตกลง',
        cancelButtonText: 'ยกเลิก'
      }).then((result) => {
        if (result.isConfirmed || !isSubmit) {
          Swal.fire({ title: 'กำลังบันทึกข้อมูล...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
          
          const payload = { assessmentId: currentAssessmentId, answers: answers };
          
          backendAPI.withSuccessHandler((response) => {
            if(response.success) {
              currentAssessmentId = response.assessmentId;
              Swal.fire('สำเร็จ!', `ข้อมูลถูกบันทึกเรียบร้อยแล้ว สถานะ: ${response.status}`, 'success').then(()=>{
                if(isSubmit) window.location.reload();
              });
            } else {
              Swal.fire('ข้อผิดพลาด', response.error, 'error');
            }
          }).saveAssessment(payload, isSubmit);
        }
      });
    }

    // 6. ระบบ Anti-Cheat พื้นฐาน
    document.addEventListener('contextmenu', event => event.preventDefault()); 
    document.addEventListener('keydown', (e) => {
      if ((e.ctrlKey && (e.key === 'c' || e.key === 'p' || e.key === 's')) || e.key === 'F12') {
        e.preventDefault(); 
      }
    });
    
    // เบลอหน้าจอเมื่อเปลี่ยนแท็บเพื่อป้องกันการค้นหาข้อมูล
    window.addEventListener('blur', () => {
      if(isMonitoring) {
        document.getElementById('app-content').style.filter = 'blur(15px)';
      }
    });
    window.addEventListener('focus', () => {
      document.getElementById('app-content').style.filter = 'none';
    });
    
  </script>
</body>
</html>
