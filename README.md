<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>HR Store Assessment 2026</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- TensorFlow.js สำหรับ AI Proctoring -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
  <!-- SweetAlert2 สำหรับแจ้งเตือนสวยงาม -->
  <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
  
  <style>
    body { background-color: #f3f4f6; font-family: 'Sarabun', sans-serif; }
    .makro-red { background-color: #990000; }
    .makro-text { color: #990000; }
    .blur-screen { filter: blur(15px); pointer-events: none; user-select: none; }
    #webcam-container { position: fixed; top: 10px; left: 10px; z-index: 50; border: 2px solid #10B981; border-radius: 8px; overflow: hidden; width: 120px; }
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
    <h1 class="text-xl font-bold md:text-2xl">HR Store Assessment 2026</h1>
    <div id="user-info" class="text-xs md:text-sm font-semibold truncate max-w-[50%]">กรุณาเข้าสู่ระบบ</div>
  </nav>

  <div class="container mx-auto p-4 md:p-6" id="app-content">
    
    <!-- หน้า Loading -->
    <div id="loading" class="text-center mt-20 hidden">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-red-800 mx-auto"></div>
      <p class="mt-4 text-gray-600 font-semibold">กำลังดำเนินการ...</p>
    </div>

    <!-- หน้าต่างล็อกอินด้วยรหัสพนักงาน -->
    <div id="login-view" class="max-w-md mx-auto bg-white p-8 rounded-lg shadow-md mt-10 border-t-4 border-red-800">
      <h2 class="text-2xl font-bold text-center mb-6 makro-text">เข้าสู่ระบบการประเมิน</h2>
      <div class="mb-4">
        <label class="block text-sm font-bold text-gray-700 mb-2">รหัสพนักงาน</label>
        <input type="text" id="emp-id-input" class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-red-600" placeholder="ระบุรหัสพนักงานของคุณ">
      </div>
      <button onclick="handleLogin()" class="w-full makro-red text-white font-bold py-2 px-4 rounded-lg hover:bg-red-900 transition duration-200 shadow-md">
        เข้าสู่ระบบ
      </button>
    </div>

    <!-- หน้าแบบประเมินหลัก -->
    <div id="employee-view" class="hidden">
      <!-- กล่องแสดงชื่อ-นามสกุลผู้ทำแบบประเมิน -->
      <div class="bg-red-50 p-4 md:p-6 rounded-lg shadow-sm mb-6 border border-red-100 flex items-center justify-between">
         <div>
            <p class="text-xs text-red-600 font-bold uppercase tracking-wider mb-1">ผู้ทำแบบประเมิน</p>
            <p class="text-xl md:text-2xl font-bold text-gray-800" id="profile-name">ชื่อ-นามสกุล</p>
            <p class="text-sm font-medium text-gray-600 mt-1" id="profile-details">รหัสพนักงาน: - | สาขา: -</p>
         </div>
         <div class="hidden md:block bg-red-800 text-white p-3 rounded-full shadow-inner">
            <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>
         </div>
      </div>

      <div class="bg-white p-6 rounded-lg shadow-md mb-6 border-t-4 border-red-800">
        <h2 class="text-lg font-bold text-gray-800 mb-2 flex items-center">
           <svg class="w-5 h-5 mr-2 text-red-700" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z" clip-rule="evenodd"></path></svg>
           คำชี้แจงการประเมิน
        </h2>
        <p class="text-sm text-gray-600 mb-4">แบบประเมินนี้ไม่ได้มุ่งวัดการจดจำหลักการ แต่ให้ความสำคัญกับประสบการณ์ วิธีดำเนินการ และผลลัพธ์จากการทำงานจริง กรุณาตอบคำถามตามหลัก <strong>STAR</strong> (Situation, Task, Action, Result) พร้อมระบุตัวเลขหรือหลักฐานประกอบเท่าที่สามารถเปิดเผยได้</p>
        
        <div class="bg-red-50 p-4 rounded border border-red-200">
          <label class="flex items-center space-x-3 cursor-pointer">
            <input type="checkbox" id="accept-policy" class="form-checkbox h-5 w-5 text-red-600 rounded" onchange="startAssessment()">
            <span class="text-sm font-semibold text-red-800 leading-snug">ข้าพเจ้ายอมรับเงื่อนไข เข้าใจว่าการประเมินนี้มีผลต่อการพัฒนาศักยภาพ และอนุญาตให้ระบบเปิดกล้องเว็บแคมเพื่อใช้ AI ป้องกันการทุจริตตลอดการทำแบบประเมิน</span>
          </label>
        </div>
      </div>

      <div id="questions-container" class="hidden space-y-6"></div>
      
      <div id="action-buttons" class="hidden mt-6 flex justify-between">
        <button onclick="saveDraft()" class="bg-gray-500 text-white px-6 py-2 rounded-lg shadow hover:bg-gray-600 font-medium">บันทึกร่าง (Draft)</button>
        <button onclick="submitAssessment()" class="makro-red text-white px-8 py-2 rounded-lg shadow-md hover:bg-red-900 font-bold text-lg">ส่งแบบประเมิน</button>
      </div>
    </div>
  </div>

  <script>
    let userContext = {};
    let questionsData = [];
    let isMonitoring = false;
    let currentAssessmentId = '';

    // Mock Object สำหรับรัน Preview ใน Editor
    const getProxy = () => {
      const p = {
        loginWithEmployeeId: (id) => {
            if(id === '00020686') return {empId: id, name: 'คุณประมวล (Admin)', role: 'Administrator', branch: 'Head Office'};
            return { empId: id, name: 'พนักงาน ทดสอบ', role: 'HR Store', branch: 'Makro สาขา 1' };
        },
        getQuestions: () => [
          { id: 'Q01', dimension: 'Leadership', text: 'เล่าปัญหาด้านทักษะที่พบ' },
          { id: 'Q02', dimension: 'Agility', text: 'เล่าปัญหาการรับมือการเปลี่ยนแปลง' }
        ],
        saveAssessment: () => ({ success: true, assessmentId: 'Mock-UUID-123' })
      };
      return new Proxy(p, {
        get: function(target, prop) {
          if (prop === 'withSuccessHandler') return function(cb) { target.cb = cb; return new Proxy(target, this); };
          if (prop === 'withFailureHandler') return function(cb) { target.err = cb; return new Proxy(target, this); };
          if (typeof target[prop] === 'function') {
            return function(...args) {
              const res = target[prop](...args);
              if (target.cb) setTimeout(() => target.cb(res), 500);
              return res;
            }
          }
          return target[prop];
        }
      });
    };

    function handleLogin() {
      const empId = document.getElementById('emp-id-input').value;
      if(!empId) {
        Swal.fire('ข้อผิดพลาด', 'กรุณากรอกรหัสพนักงาน', 'warning');
        return;
      }
      
      document.getElementById('login-view').classList.add('hidden');
      document.getElementById('loading').classList.remove('hidden');

      const server = typeof google !== 'undefined' ? google.script.run : getProxy();
      server.withSuccessHandler(initApp).withFailureHandler(e => {
        Swal.fire('Error', e.message, 'error');
        document.getElementById('loading').classList.add('hidden');
        document.getElementById('login-view').classList.remove('hidden');
      }).loginWithEmployeeId(empId);
    }

    function initApp(user) {
      userContext = user;
      document.getElementById('loading').classList.add('hidden');
      
      if(!user || user.role === 'None') {
        Swal.fire({ icon: 'error', title: 'เข้าสู่ระบบไม่สำเร็จ', text: user ? user.error : 'ไม่พบข้อมูล' })
          .then(() => document.getElementById('login-view').classList.remove('hidden'));
        return;
      }
      
      // แสดงชื่อ และ สาขา ให้ชัดเจน
      document.getElementById('user-info').innerText = `${user.name} | สาขา: ${user.branch} | ${user.role}`;
      
      // อัปเดตข้อมูลในกล่อง Profile ใหญ่
      document.getElementById('profile-name').innerText = user.name;
      document.getElementById('profile-details').innerText = `รหัสพนักงาน: ${user.empId} | ตำแหน่ง: ${user.role} | สาขา: ${user.branch}`;
      
      // ป้องกันพิมพ์ Role ผิดพลาด
      const safeRole = String(user.role).trim().toLowerCase();
      
      if(safeRole === 'hr store' || safeRole === 'administrator') {
        document.getElementById('employee-view').classList.remove('hidden');
        
        document.getElementById('loading').classList.remove('hidden');
        const server = typeof google !== 'undefined' ? google.script.run : getProxy();
        server.withSuccessHandler((qs) => {
           document.getElementById('loading').classList.add('hidden');
           renderQuestions(qs);
        }).getQuestions();
      } else {
        Swal.fire('เข้าสู่ระบบสำเร็จ', 'แต่สิทธิ์ของคุณไม่ใช่ HR Store ไม่สามารถทำแบบประเมินได้', 'info');
      }
    }

    function renderQuestions(qs) {
      questionsData = qs;
      const container = document.getElementById('questions-container');
      let html = '';
      qs.forEach((q, idx) => {
        html += `
          <div class="bg-white p-6 rounded-lg shadow border border-gray-200" id="card-${q.id}">
            <h3 class="font-bold text-lg makro-text mb-2">ข้อ ${idx+1}: ${q.dimension}</h3>
            <p class="text-sm bg-gray-50 p-3 rounded mb-4 text-gray-700">${q.text}</p>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div>
                 <label class="text-xs font-bold text-gray-500 mb-1 block">S: สถานการณ์ (Situation)</label>
                 <textarea id="s-${q.id}" class="w-full border border-gray-300 rounded p-2 text-sm h-24 focus:ring-red-500 focus:border-red-500 outline-none transition" placeholder="ระบุสถานการณ์ที่เกิดขึ้น (ขั้นต่ำ 80 ตัวอักษร)"></textarea>
              </div>
              <div>
                 <label class="text-xs font-bold text-gray-500 mb-1 block">T: บทบาท (Task)</label>
                 <textarea id="t-${q.id}" class="w-full border border-gray-300 rounded p-2 text-sm h-24 focus:ring-red-500 focus:border-red-500 outline-none transition" placeholder="หน้าที่และความรับผิดชอบของคุณ (ขั้นต่ำ 50 ตัวอักษร)"></textarea>
              </div>
              <div>
                 <label class="text-xs font-bold text-gray-500 mb-1 block">A: การกระทำ (Action)</label>
                 <textarea id="a-${q.id}" class="w-full border border-gray-300 rounded p-2 text-sm h-24 focus:ring-red-500 focus:border-red-500 outline-none transition" placeholder="สิ่งที่คุณลงมือทำและวิธีคิด (ขั้นต่ำ 120 ตัวอักษร)"></textarea>
              </div>
              <div>
                 <label class="text-xs font-bold text-gray-500 mb-1 block">R: ผลลัพธ์ (Result)</label>
                 <textarea id="r-${q.id}" class="w-full border border-gray-300 rounded p-2 text-sm h-24 focus:ring-red-500 focus:border-red-500 outline-none transition" placeholder="ผลลัพธ์ที่เกิดขึ้น (ขั้นต่ำ 80 ตัวอักษร)"></textarea>
              </div>
            </div>
          </div>
        `;
      });
      container.innerHTML = html;
    }

    async function startAssessment() {
      if(!document.getElementById('accept-policy').checked) return;
      document.getElementById('accept-policy').disabled = true;

      // พยายามเปิดกล้อง (บังคับกล้องหน้า)
      try {
        Swal.fire({ title: 'กำลังโหลดระบบรักษาความปลอดภัย...', text: 'กรุณาอนุญาตให้ระบบเข้าถึงกล้องเว็บแคมของคุณ', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
        const video = document.getElementById('video');
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
        video.srcObject = stream;
        
        // รอโหลด AI โมเดล
        const model = await cocoSsd.load();
        Swal.close();
        
        document.getElementById('webcam-container').classList.remove('hidden');
        document.getElementById('questions-container').classList.remove('hidden');
        document.getElementById('action-buttons').classList.remove('hidden');
        
        isMonitoring = true;
        detectCheating(video, model);

      } catch (err) {
        Swal.close();
        console.error("Camera error:", err);
        Swal.fire({
          icon: 'error',
          title: 'ไม่สามารถเปิดกล้องได้',
          html: `<p class="text-sm text-gray-600">เบราว์เซอร์หรือระบบปฏิบัติการ (Windows/Mac) ปฏิเสธการเข้าถึงกล้อง กรุณากดอนุญาตที่แถบ URL หรือตรวจสอบการตั้งค่าของเครื่อง</p>
                 <div class="mt-4 pt-4 border-t border-gray-200">
                    <button id="bypass-btn" class="bg-gray-200 text-gray-700 px-4 py-2 rounded text-sm hover:bg-gray-300">คลิกที่นี่เพื่อข้ามการใช้กล้อง (Bypass สำหรับตรวจสอบระบบ)</button>
                 </div>`,
          showConfirmButton: true,
          confirmButtonText: 'รับทราบ (ลองใหม่)'
        });
        
        // ฟังก์ชัน Bypass สำหรับให้แอดมินหรือช่วงทดสอบระบบไปต่อได้
        document.getElementById('accept-policy').checked = false;
        document.getElementById('accept-policy').disabled = false;
        
        setTimeout(() => {
           const bypassBtn = document.getElementById('bypass-btn');
           if(bypassBtn) {
             bypassBtn.onclick = () => {
               Swal.close();
               document.getElementById('accept-policy').checked = true;
               document.getElementById('accept-policy').disabled = true;
               document.getElementById('questions-container').classList.remove('hidden');
               document.getElementById('action-buttons').classList.remove('hidden');
               Swal.fire('ข้ามการตรวจจับด้วยกล้อง', 'ระบบเปิดให้ทำข้อสอบแล้ว', 'info');
             };
           }
        }, 100);
      }
    }

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
      setTimeout(() => detectCheating(video, model), 2000); // วนลูปทุก 2 วินาที
    }

    function gatherAnswers() {
      let answers = [];
      let isComplete = true;
      
      questionsData.forEach(q => {
        const s = document.getElementById(`s-${q.id}`).value.trim();
        const t = document.getElementById(`t-${q.id}`).value.trim();
        const a = document.getElementById(`a-${q.id}`).value.trim();
        const r = document.getElementById(`r-${q.id}`).value.trim();
        
        if(s.length < 10 || t.length < 10 || a.length < 10 || r.length < 10) isComplete = false;
        answers.push({ questionId: q.id, dimension: q.dimension, s: s, t: t, a: a, r: r });
      });
      return { answers, isComplete };
    }

    function saveDraft() {
      const data = gatherAnswers();
      Swal.fire({ title: 'กำลังบันทึก...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
      
      const payload = {
        assessmentId: currentAssessmentId,
        user: userContext,
        answers: data.answers
      };

      const server = typeof google !== 'undefined' ? google.script.run : getProxy();
      server.withSuccessHandler((res) => {
        if(res.success) {
           currentAssessmentId = res.assessmentId;
           Swal.fire('บันทึกร่างสำเร็จ', 'คุณสามารถกลับมาทำต่อได้ในภายหลัง', 'success');
        } else {
           Swal.fire('เกิดข้อผิดพลาด', res.error, 'error');
        }
      }).saveAssessment(payload, false);
    }

    function submitAssessment() {
      const data = gatherAnswers();
      if (!data.isComplete) {
        Swal.fire('ข้อมูลไม่ครบถ้วน', 'กรุณาตอบคำถามให้ครบทุกข้อ (อย่างน้อยช่องละ 10-20 ตัวอักษร) ก่อนส่งประเมิน', 'warning');
        return;
      }
      
      Swal.fire({
        title: 'ยืนยันการส่งแบบประเมิน?',
        text: "หากส่งแล้วจะไม่สามารถแก้ไขได้จนกว่าหัวหน้าจะส่งกลับมา",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#990000',
        confirmButtonText: 'ยืนยันการส่ง',
        cancelButtonText: 'ยกเลิก'
      }).then((result) => {
        if (result.isConfirmed) {
          Swal.fire({ title: 'กำลังส่งข้อมูล...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
          
          const payload = { assessmentId: currentAssessmentId, user: userContext, answers: data.answers };
          const server = typeof google !== 'undefined' ? google.script.run : getProxy();
          
          server.withSuccessHandler((res) => {
            if(res.success) {
               Swal.fire('ส่งสำเร็จ!', 'ระบบได้รับแบบประเมินของคุณเรียบร้อยแล้ว', 'success').then(() => {
                 location.reload();
               });
            } else {
               Swal.fire('เกิดข้อผิดพลาด', res.error, 'error');
            }
          }).saveAssessment(payload, true);
        }
      });
    }

    document.addEventListener('contextmenu', event => event.preventDefault()); 
    document.addEventListener('keydown', (e) => {
      if ((e.ctrlKey && (e.key === 'c' || e.key === 'p' || e.key === 's')) || e.key === 'F12') e.preventDefault(); 
    });
    window.addEventListener('blur', () => { if(isMonitoring) document.getElementById('app-content').style.filter = 'blur(15px)'; });
    window.addEventListener('focus', () => { document.getElementById('app-content').style.filter = 'none'; });

  </script>
</body>
</html>
