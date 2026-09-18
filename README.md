<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Check-In Online - ระบบเช็คชื่อนักเรียนข้ามอุปกรณ์และข้ามเครือข่าย</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts (Prompt) -->
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        prompt: ['Prompt', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            500: '#0284c7',
                            600: '#0284c7',
                            700: '#0369a1',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body { 
            font-family: 'Prompt', sans-serif; 
        }
        .glass-card {
            background: rgba(255, 255, 255, 0.98);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(226, 232, 240, 0.8);
        }
        .pulse-slow {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: .5; }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col selection:bg-blue-500 selection:text-white">

    <!-- Navigation Bar (Hidden in Student Mode) -->
    <header id="main-header" class="bg-slate-900 text-white shadow-lg sticky top-0 z-40 border-b border-slate-800">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-3 cursor-pointer" onclick="switchTab('dashboard')">
                <div class="bg-gradient-to-tr from-blue-600 to-indigo-600 p-2.5 rounded-xl text-white shadow-md shadow-blue-500/20 flex items-center justify-center">
                    <i class="fa-solid fa-qrcode text-xl"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight flex items-center">
                        Smart Check-In
                        <span class="ml-2 text-[10px] bg-blue-500/20 text-blue-400 border border-blue-500/30 font-semibold px-2 py-0.5 rounded-full">v3.4 Sync Fixed</span>
                    </h1>
                    <p class="text-xs text-slate-400">ระบบเช็คชื่อข้ามอุปกรณ์และข้ามเครือข่าย Real-time Cloud</p>
                </div>
            </div>
            
            <div id="teacher-nav" class="hidden md:flex space-x-1 text-sm font-medium">
                <button onclick="switchTab('dashboard')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-blue-400" id="nav-dashboard">
                    <i class="fa-solid fa-chalkboard-user mr-1.5"></i>เปิดคาบเรียน
                </button>
                <button onclick="switchTab('classes')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-classes">
                    <i class="fa-solid fa-school mr-1.5"></i>จัดการห้องเรียน
                </button>
                <button onclick="switchTab('students')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-students">
                    <i class="fa-solid fa-users mr-1.5"></i>จัดการนักเรียน
                </button>
                <button onclick="switchTab('reports')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-reports">
                    <i class="fa-solid fa-chart-line mr-1.5"></i>รายงานสถิติ
                </button>
            </div>

            <div class="flex items-center space-x-2">
                <span id="sync-status" class="inline-flex items-center text-xs px-3 py-1.5 rounded-full bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 font-medium">
                    <span class="w-2 h-2 rounded-full bg-emerald-400 mr-1.5 pulse-slow"></span>
                    <span>Cloud Sync Active</span>
                </span>
            </div>
        </div>
        
        <!-- Mobile Navigation Menu -->
        <div id="teacher-nav-mobile" class="md:hidden flex justify-around border-t border-slate-800 py-2 text-xs bg-slate-900/95 backdrop-blur">
            <button onclick="switchTab('dashboard')" class="text-blue-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-chalkboard-user text-base mb-1"></i>เช็คชื่อ
            </button>
            <button onclick="switchTab('classes')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-school text-base mb-1"></i>ห้องเรียน
            </button>
            <button onclick="switchTab('students')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-users text-base mb-1"></i>นักเรียน
            </button>
            <button onclick="switchTab('reports')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-chart-line text-base mb-1"></i>รายงาน
            </button>
        </div>
    </header>

    <!-- Main Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

        <!-- STUDENT VIEW -->
        <div id="student-view" class="hidden max-w-md mx-auto py-4">
            <div class="text-center mb-6">
                <div class="inline-flex p-3 bg-blue-600 text-white rounded-2xl shadow-lg shadow-blue-500/30 mb-2">
                    <i class="fa-solid fa-qrcode text-3xl"></i>
                </div>
                <h1 class="text-xl font-bold text-slate-800">ระบบเช็คชื่อนักเรียนออนไลน์</h1>
                <p class="text-xs text-slate-500">เลือกชื่อของคุณและขอรับรหัส OTP เพื่อเช็คชื่อ</p>
            </div>

            <div class="glass-card rounded-2xl shadow-xl p-6 text-center border-t-4 border-blue-600 relative overflow-hidden">
                <div class="w-16 h-16 bg-blue-50 text-blue-600 rounded-2xl flex items-center justify-center mx-auto mb-4 text-2xl shadow-inner border border-blue-100">
                    <i class="fa-solid fa-user-check"></i>
                </div>
                <h2 class="text-xl font-bold text-slate-800" id="student-class-title">กำลังดึงข้อมูลคาบเรียน...</h2>
                <p class="text-xs text-slate-500 mt-1 mb-6 font-medium" id="student-session-info">กำลังเชื่อมต่อฐานข้อมูล Cloud...</p>

                <!-- Student Step 1: Select Student Name -->
                <div id="student-step-select" class="space-y-4">
                    <div class="text-left">
                        <div class="flex justify-between items-center mb-2">
                            <label class="block text-xs font-bold text-slate-700 uppercase flex items-center">
                                <span class="w-5 h-5 rounded-full bg-blue-600 text-white flex items-center justify-center text-[10px] mr-1.5">1</span>
                                เลือกชื่อ-นามสกุลของคุณ
                            </label>
                            <button onclick="fetchStudentSessionData(true)" class="text-[11px] text-blue-600 hover:text-blue-800 font-semibold flex items-center">
                                <i class="fa-solid fa-arrows-rotate mr-1"></i> โหลดใหม่
                            </button>
                        </div>
                        <select id="student-dropdown" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3.5 text-slate-800 font-medium focus:ring-2 focus:ring-blue-500 focus:outline-none shadow-sm text-sm">
                            <option value="">-- กำลังโหลดรายชื่อนักเรียน --</option>
                        </select>
                    </div>

                    <button onclick="requestStudentOTP()" class="w-full bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 active:scale-95 text-white font-semibold py-3.5 px-4 rounded-xl shadow-lg shadow-blue-500/30 transition duration-200 flex items-center justify-center space-x-2 text-sm">
                        <i class="fa-solid fa-key"></i>
                        <span>ขอรับรหัส OTP 6 หลัก</span>
                    </button>
                </div>

                <!-- Student Step 2: Show OTP Code -->
                <div id="student-step-otp" class="hidden mt-4 bg-slate-900 text-white rounded-2xl p-6 shadow-xl relative overflow-hidden text-center">
                    <span class="inline-flex items-center bg-amber-500/20 text-amber-300 text-xs px-3 py-1 rounded-full font-medium mb-3 border border-amber-500/30">
                        <i class="fa-solid fa-circle-notch animate-spin mr-1.5"></i>รอคุณครูอนุมัติรหัส
                    </span>
                    <p class="text-xs text-slate-400">แจ้งรหัส 6 หลักนี้แก่คุณครูเพื่อยืนยัน:</p>
                    <div class="text-4xl font-black tracking-widest text-amber-400 my-3 font-mono drop-shadow-md" id="display-otp-code">
                        ------
                    </div>
                    <div class="bg-slate-800/90 p-3 rounded-xl border border-slate-700 text-xs text-slate-300 leading-relaxed text-left space-y-1">
                        <p class="font-semibold text-slate-200"><i class="fa-solid fa-circle-info text-amber-400 mr-1"></i> คำแนะนำ:</p>
                        <p>1. บอกรหัสนี้แก่คุณครูเพื่ออนุมัติ</p>
                        <p>2. หรือเมื่อครูกดอนุมัติผ่านคอมพิวเตอร์ หน้าจอนี้จะเปลี่ยนเป็นสีเขียวทันที</p>
                    </div>
                </div>

                <!-- Student Step 3: Success State -->
                <div id="student-step-success" class="hidden mt-4 bg-emerald-50 text-emerald-900 rounded-2xl p-6 border border-emerald-200 shadow-md text-center space-y-2">
                    <div class="w-16 h-16 bg-emerald-500 text-white rounded-full flex items-center justify-center mx-auto text-2xl shadow-lg shadow-emerald-500/30">
                        <i class="fa-solid fa-check text-3xl"></i>
                    </div>
                    <h3 class="font-bold text-2xl text-emerald-800">เช็คชื่อสำเร็จแล้ว! 🟢</h3>
                    <p class="text-xs text-emerald-700 font-medium">คุณครูยืนยันการเข้าเรียนเรียบร้อยแล้ว ขอบคุณครับ</p>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 1 - LIVE DASHBOARD -->
        <div id="tab-dashboard" class="tab-content space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 items-end">
                    <div>
                        <label class="block text-xs font-bold text-slate-600 mb-1.5 uppercase tracking-wide">1. เลือกห้องเรียน</label>
                        <select id="teacher-class-select" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 text-slate-800 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                            <option value="">-- เลือกห้องเรียน --</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-600 mb-1.5 uppercase tracking-wide">2. ชื่อวิชา / คาบเรียน</label>
                        <input type="text" id="teacher-subject-input" placeholder="เช่น วิทยาการคำนวณ คาบ 1" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 text-slate-800 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                    </div>
                    <div>
                        <button id="btn-toggle-session" onclick="toggleClassSession()" class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm">
                            <i class="fa-solid fa-play"></i>
                            <span>🟢 เริ่มเปิดคาบเรียน (Cloud Sync)</span>
                        </button>
                    </div>
                </div>
            </div>

            <div id="active-session-container" class="hidden grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-1 space-y-6">
                    <div class="glass-card rounded-2xl p-6 shadow-sm text-center border-t-4 border-blue-600">
                        <span class="bg-emerald-100 text-emerald-800 text-[11px] px-3 py-1 rounded-full font-bold inline-block mb-3 border border-emerald-200 shadow-sm">
                            🔴 LIVE CROSS-NETWORK
                        </span>
                        <h3 class="font-bold text-slate-800 text-xl" id="live-class-name">ห้องเรียน</h3>
                        <p class="text-xs text-slate-500 mb-4 font-medium" id="live-subject-name">วิชา</p>
                        
                        <div class="bg-white p-4 rounded-2xl shadow-inner inline-block border border-slate-200">
                            <div id="qrcode" class="flex justify-center"></div>
                        </div>
                        <p class="text-xs text-slate-500 mt-3 font-medium">ให้นักเรียนสแกน QR Code เพื่อเลือกชื่อและขอ OTP</p>
                        
                        <div class="mt-4 pt-4 border-t border-slate-100">
                            <input type="text" id="session-link-input" readonly class="w-full text-xs bg-slate-100 border border-slate-200 rounded-lg p-2.5 text-slate-600 font-mono text-center mb-2 focus:outline-none select-all">
                            <button onclick="copySessionLink()" class="w-full text-xs bg-blue-50 hover:bg-blue-100 text-blue-600 font-semibold py-2.5 px-3 rounded-xl border border-blue-200 transition flex items-center justify-center space-x-1.5 active:scale-95">
                                <i class="fa-solid fa-copy"></i>
                                <span>คัดลอกลิงก์ให้นักเรียน</span>
                            </button>
                        </div>
                    </div>

                    <div class="glass-card rounded-2xl p-6 shadow-md bg-slate-900 text-white">
                        <h4 class="font-bold text-base mb-1 flex items-center">
                            <i class="fa-solid fa-shield-halved text-amber-400 mr-2"></i> กรอก OTP ยืนยันให้เด็ก
                        </h4>
                        <p class="text-xs text-slate-400 mb-4">พิมพ์รหัส 6 หลักที่นักเรียนแจ้งเพื่อยืนยันเข้าเรียนทันที</p>
                        <div class="flex space-x-2">
                            <input type="text" id="teacher-otp-input" placeholder="000000" maxlength="6" class="w-full text-center tracking-widest font-mono text-2xl font-bold bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-amber-400 focus:outline-none focus:ring-2 focus:ring-amber-400">
                            <button onclick="verifyTeacherOTP()" class="bg-amber-500 hover:bg-amber-600 active:scale-95 text-slate-950 font-bold px-5 rounded-xl transition shadow-lg flex items-center justify-center shrink-0">
                                ยืนยัน
                            </button>
                        </div>
                    </div>
                </div>

                <div class="lg:col-span-2">
                    <div class="glass-card rounded-2xl p-6 shadow-sm min-h-full flex flex-col justify-between">
                        <div>
                            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-2">
                                <div>
                                    <h3 class="font-bold text-lg text-slate-800 flex items-center">
                                        ตารางเช็คชื่อ Real-time
                                        <span class="ml-2 w-2.5 h-2.5 rounded-full bg-emerald-500 pulse-slow"></span>
                                    </h3>
                                    <p class="text-xs text-slate-500">ข้อมูลอัปเดตซิงค์ผ่าน Cloud อัตโนมัติทุกๆ 1.5 วินาที</p>
                                </div>
                                <div class="flex space-x-2 text-xs font-semibold">
                                    <span class="px-3 py-1.5 rounded-lg bg-red-50 text-red-700 border border-red-200">🔴 ยังไม่เช็ค: <span id="cnt-absent">0</span></span>
                                    <span class="px-3 py-1.5 rounded-lg bg-amber-50 text-amber-800 border border-amber-200">🟡 รออนุมัติ: <span id="cnt-pending">0</span></span>
                                    <span class="px-3 py-1.5 rounded-lg bg-emerald-50 text-emerald-800 border border-emerald-200">🟢 เข้าเรียน: <span id="cnt-present">0</span></span>
                                </div>
                            </div>

                            <div class="overflow-x-auto rounded-xl border border-slate-200 shadow-sm">
                                <table class="w-full text-left border-collapse text-sm">
                                    <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase tracking-wider">
                                        <tr>
                                            <th class="p-3.5 border-b">เลขที่</th>
                                            <th class="p-3.5 border-b">รหัสนักเรียน</th>
                                            <th class="p-3.5 border-b">ชื่อ - นามสกุล</th>
                                            <th class="p-3.5 border-b text-center">สถานะ</th>
                                            <th class="p-3.5 border-b text-center">OTP</th>
                                            <th class="p-3.5 border-b text-center">จัดการ</th>
                                        </tr>
                                    </thead>
                                    <tbody id="live-students-tbody" class="divide-y divide-slate-100 bg-white">
                                        <tr>
                                            <td colspan="6" class="text-center py-8 text-slate-400">กำลังดึงข้อมูลนักเรียนจาก Cloud...</td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <div class="mt-4 pt-4 border-t border-slate-100 flex justify-between items-center text-xs text-slate-400">
                            <span>Cloud Service Status: Online</span>
                            <span class="font-mono text-emerald-600 font-medium"><i class="fa-solid fa-arrows-rotate animate-spin mr-1"></i> Live Polling Active</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 2 - CLASS MANAGEMENT -->
        <div id="tab-classes" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <h3 class="font-bold text-lg mb-4 text-slate-800">เพิ่มห้องเรียนใหม่</h3>
                <form id="form-add-class" onsubmit="addClass(event)" class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                    <input type="text" id="input-class-name" placeholder="ชื่อห้องเรียน (เช่น ม.4/1)" required class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                    <input type="text" id="input-class-desc" placeholder="คำอธิบายเพิ่มเติม (ตัวอย่าง: ปีการศึกษา 2569)" class="bg-slate-50 border border-slate-300 rounded-xl p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium">
                    <button type="submit" class="bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow transition duration-200 text-sm">
                        <i class="fa-solid fa-plus mr-1"></i> เพิ่มห้องเรียน
                    </button>
                </form>
            </div>

            <div class="glass-card rounded-2xl p-6 shadow-sm">
                <h3 class="font-bold text-lg mb-4 text-slate-800">รายชื่อห้องเรียนทั้งหมด</h3>
                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase tracking-wider">
                            <tr>
                                <th class="p-3.5 border-b">ลำดับ</th>
                                <th class="p-3.5 border-b">ชื่อห้องเรียน</th>
                                <th class="p-3.5 border-b">คำอธิบาย</th>
                                <th class="p-3.5 border-b text-center">จำนวนนักเรียน</th>
                                <th class="p-3.5 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="classes-table-tbody" class="divide-y divide-slate-100 bg-white">
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 3 - STUDENT MANAGEMENT -->
        <div id="tab-students" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 class="font-bold text-lg mb-4 text-slate-800">เพิ่มนักเรียนรายบุคคล</h3>
                    <form onsubmit="addSingleStudent(event)" class="space-y-3">
                        <div>
                            <label class="block text-xs font-bold text-slate-600 mb-1">ห้องเรียน</label>
                            <select id="single-std-class" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-medium">
                                <option value="">-- เลือกห้องเรียน --</option>
                            </select>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-bold text-slate-600 mb-1">เลขที่</label>
                                <input type="number" id="single-std-no" required placeholder="1" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-medium">
                            </div>
                            <div>
                                <label class="block text-xs font-bold text-slate-600 mb-1">รหัสนักเรียน</label>
                                <input type="text" id="single-std-id" required placeholder="10001" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-medium">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-600 mb-1">ชื่อ - นามสกุล</label>
                            <input type="text" id="single-std-name" required placeholder="นายสมชาย ใจดี" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-medium">
                        </div>
                        <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold py-2.5 rounded-xl transition shadow text-sm">
                            <i class="fa-solid fa-user-plus mr-1"></i> บันทึกนักเรียน
                        </button>
                    </form>
                </div>

                <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 class="font-bold text-lg mb-2 text-slate-800">นำเข้ารายชื่อหลายคน (Bulk Import)</h3>
                    <p class="text-xs text-slate-500 mb-3">คัดลอกรายชื่อจาก Excel ในรูปแบบ: <code>เลขที่[Tab]รหัส[Tab]ชื่อ-นามสกุล</code></p>
                    <form onsubmit="addBulkStudents(event)" class="space-y-3">
                        <div>
                            <label class="block text-xs font-bold text-slate-600 mb-1">ห้องเรียน</label>
                            <select id="bulk-std-class" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-medium">
                                <option value="">-- เลือกห้องเรียน --</option>
                            </select>
                        </div>
                        <div>
                            <textarea id="bulk-std-text" rows="5" placeholder="1	10001	นายสมชาย ใจดี&#10;2	10002	นางสาวสมหญิง มีสุข" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-mono"></textarea>
                        </div>
                        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold py-2.5 rounded-xl transition shadow text-sm">
                            <i class="fa-solid fa-file-import mr-1"></i> นำเข้าข้อมูลรายชื่อ
                        </button>
                    </form>
                </div>
            </div>

            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-2">
                    <h3 class="font-bold text-lg text-slate-800">รายชื่อนักเรียนในระบบ</h3>
                    <select id="filter-std-class" onchange="renderStudentsList()" class="bg-slate-50 border border-slate-300 rounded-xl p-2 text-sm font-medium">
                        <option value="ALL">-- แสดงทุกห้องเรียน --</option>
                    </select>
                </div>
                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase tracking-wider">
                            <tr>
                                <th class="p-3.5 border-b">ห้องเรียน</th>
                                <th class="p-3.5 border-b">เลขที่</th>
                                <th class="p-3.5 border-b">รหัสนักเรียน</th>
                                <th class="p-3.5 border-b">ชื่อ - นามสกุล</th>
                                <th class="p-3.5 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="students-table-tbody" class="divide-y divide-slate-100 bg-white">
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 4 - REPORTS -->
        <div id="tab-reports" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 gap-4">
                    <div>
                        <h3 class="font-bold text-lg text-slate-800">รายงานประวัติการเช็คชื่อย้อนหลัง</h3>
                        <p class="text-xs text-slate-500">ค้นหาและส่งออกประวัติ session การเข้าเรียน</p>
                    </div>
                    <button onclick="exportReportsToCSV()" class="bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-medium px-4 py-2.5 rounded-xl text-sm transition shadow flex items-center">
                        <i class="fa-solid fa-file-excel mr-2"></i> ส่งออกเป็นไฟล์ CSV (Excel)
                    </button>
                </div>

                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-700 font-bold text-xs uppercase tracking-wider">
                            <tr>
                                <th class="p-3.5 border-b">วัน-เวลา</th>
                                <th class="p-3.5 border-b">ห้องเรียน</th>
                                <th class="p-3.5 border-b">วิชา</th>
                                <th class="p-3.5 border-b text-center">มาเรียน</th>
                                <th class="p-3.5 border-b text-center">ขาดเรียน</th>
                                <th class="p-3.5 border-b text-center">สถานะ</th>
                            </tr>
                        </thead>
                        <tbody id="reports-table-tbody" class="divide-y divide-slate-100 bg-white">
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- Application Script -->
    <script>
        // Exact Root Endpoint of Realtime Database
        const CLOUD_DB_BASE_URL = "https://checkin-realtime-default-rtdb.asia-southeast1.firebasedatabase.app";

        let pollingInterval = null;
        let currentSessionId = null;
        let currentSessionData = null;
        let currentStudentSelectedId = null;

        window.appState = {
            classes: JSON.parse(localStorage.getItem('sc_classes')) || [
                { id: 'c1', name: 'ม.4/1', desc: 'สายวิทยาศาสตร์-คณิตศาสตร์' },
                { id: 'c2', name: 'ม.4/2', desc: 'สายภาษา-สังคม' }
            ],
            students: JSON.parse(localStorage.getItem('sc_students')) || [
                { id: 's1', classId: 'c1', no: 1, stdId: '10001', name: 'นายกิตติพงษ์ วงศ์สว่าง' },
                { id: 's2', classId: 'c1', no: 2, stdId: '10002', name: 'นางสาวจิราพร แสงทอง' },
                { id: 's3', classId: 'c1', no: 3, stdId: '10003', name: 'นายธนกร รัตนเดช' },
                { id: 's4', classId: 'c2', no: 1, stdId: '20001', name: 'นางสาวปรียาพร พรหมมา' }
            ],
            history: JSON.parse(localStorage.getItem('sc_history')) || []
        };

        function saveLocalState() {   
            localStorage.setItem('sc_classes', JSON.stringify(window.appState.classes));
            localStorage.setItem('sc_students', JSON.stringify(window.appState.students));
            localStorage.setItem('sc_history', JSON.stringify(window.appState.history));
        }

        function getSessionIdFromUrl() {
            const urlParams = new URLSearchParams(window.location.search);
            let sessionId = urlParams.get('session');
            if (!sessionId && window.location.hash) {
                const hashParams = new URLSearchParams(window.location.hash.replace('#', '?'));
                sessionId = hashParams.get('session');
            }
            return sessionId;
        }

        window.addEventListener('DOMContentLoaded', () => {
            const sessionIdParam = getSessionIdFromUrl();

            if (sessionIdParam) {
                // STUDENT MODE
                currentSessionId = sessionIdParam;
                document.getElementById('main-header').classList.add('hidden');
                document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
                document.getElementById('student-view').classList.remove('hidden');
                
                fetchStudentSessionData();
                pollingInterval = setInterval(fetchStudentSessionData, 2000);
            } else {
                // TEACHER MODE
                populateClassDropdowns();
                renderClassesList();
                renderStudentsList();
                renderReportsList();
                switchTab('dashboard');
            }
        });

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            const target = document.getElementById(`tab-${tabId}`);
            if (target) target.classList.remove('hidden');

            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.classList.remove('text-blue-400', 'bg-slate-800');
                btn.classList.add('text-slate-300');
            });
            const activeNav = document.getElementById(`nav-${tabId}`);
            if (activeNav) {
                activeNav.classList.remove('text-slate-300');
                activeNav.classList.add('text-blue-400', 'bg-slate-800');
            }
        }

        function populateClassDropdowns() {
            const selectTeacher = document.getElementById('teacher-class-select');
            const selectSingle = document.getElementById('single-std-class');
            const selectBulk = document.getElementById('bulk-std-class');
            const selectFilter = document.getElementById('filter-std-class');

            const optionsHtml = window.appState.classes.map(c => `<option value="${c.id}">${c.name}</option>`).join('');
            
            if (selectTeacher) selectTeacher.innerHTML = '<option value="">-- เลือกห้องเรียน --</option>' + optionsHtml;
            if (selectSingle) selectSingle.innerHTML = '<option value="">-- เลือกห้องเรียน --</option>' + optionsHtml;
            if (selectBulk) selectBulk.innerHTML = '<option value="">-- เลือกห้องเรียน --</option>' + optionsHtml;
            if (selectFilter) selectFilter.innerHTML = '<option value="ALL">-- แสดงทุกห้องเรียน --</option>' + optionsHtml;
        }

        async function toggleClassSession() {
            const btn = document.getElementById('btn-toggle-session');
            const classSelect = document.getElementById('teacher-class-select');
            const subjectInput = document.getElementById('teacher-subject-input');

            if (!currentSessionId) {
                const classId = classSelect.value;
                const subject = subjectInput.value.trim() || 'เช็คชื่อเข้าเรียน';
                if (!classId) {
                    Swal.fire({ icon: 'warning', title: 'โปรดเลือกห้องเรียน', text: 'กรุณาเลือกห้องเรียนที่ต้องการเปิดเช็คชื่อ' });
                    return;
                }

                const selectedClass = window.appState.classes.find(c => c.id === classId);
                const classStudents = window.appState.students.filter(s => s.classId === classId);

                if (classStudents.length === 0) {
                    Swal.fire({ icon: 'warning', title: 'ไม่พบนักเรียน', text: 'ห้องเรียนนี้ยังไม่มีรายชื่อนักเรียน กรุณาเพิ่มนักเรียนก่อน' });
                    return;
                }

                currentSessionId = 'S_' + Date.now();
                const sessionStudents = {};
                classStudents.forEach(s => {
                    sessionStudents[s.id] = {
                        id: s.id,
                        no: s.no,
                        stdId: s.stdId,
                        name: s.name,
                        status: 'absent',
                        otp: ''
                    };
                });

                const payload = {
                    sessionId: currentSessionId,
                    classId: selectedClass.id,
                    className: selectedClass.name,
                    subject: subject,
                    timestamp: new Date().toISOString(),
                    active: true,
                    students: sessionStudents
                };

                try {
                    await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`, {
                        method: 'PUT',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    btn.className = "w-full bg-rose-600 hover:bg-rose-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm";
                    btn.innerHTML = `<i class="fa-solid fa-stop"></i><span>🔴 ปิดคาบเรียน (บันทึกรายงาน)</span>`;
                    classSelect.disabled = true;
                    subjectInput.disabled = true;

                    document.getElementById('active-session-container').classList.remove('hidden');
                    document.getElementById('live-class-name').textContent = selectedClass.name;
                    document.getElementById('live-subject-name').textContent = subject;

                    const baseUrl = window.location.origin + window.location.pathname;
                    const studentUrl = `${baseUrl}?session=${currentSessionId}`;
                    document.getElementById('session-link-input').value = studentUrl;
                    
                    document.getElementById('qrcode').innerHTML = '';
                    new QRCode(document.getElementById("qrcode"), {
                        text: studentUrl,
                        width: 160,
                        height: 160,
                        colorDark : "#0f172a",
                        colorLight : "#ffffff",
                        correctLevel : QRCode.CorrectLevel.H
                    });

                    fetchTeacherSessionData();
                    pollingInterval = setInterval(fetchTeacherSessionData, 1500);

                    Swal.fire({ icon: 'success', title: 'เปิดคาบเรียนสำเร็จ', text: 'ระบบกำลังซิงค์ข้อมูลผ่าน Cloud สามารถให้นักเรียนสแกนได้ทันที', timer: 1800, showConfirmButton: false });
                } catch (e) {
                    Swal.fire({ icon: 'error', title: 'เชื่อมต่อ Cloud ล้มเหลว', text: 'ไม่สามารถสร้าง Session บน Cloud ได้: ' + e.message });
                    currentSessionId = null;
                }

            } else {
                Swal.fire({
                    title: 'ต้องการปิดคาบเรียนหรือไม่?',
                    text: "ระบบจะทำการสรุปผลและบันทึกประวัติเข้าสู่ระบบรายงาน",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#0284c7',
                    cancelButtonColor: '#64748b',
                    confirmButtonText: 'ยืนยันปิดคาบเรียน',
                    cancelButtonText: 'ยกเลิก'
                }).then(async (result) => {
                    if (result.isConfirmed) {
                        clearInterval(pollingInterval);
                        
                        if (currentSessionData) {
                            const studentsArr = Object.values(currentSessionData.students || {});
                            const presentCount = studentsArr.filter(s => s.status === 'present').length;
                            const absentCount = studentsArr.length - presentCount;

                            window.appState.history.unshift({
                                id: currentSessionData.sessionId,
                                timestamp: new Date().toLocaleString('th-TH'),
                                className: currentSessionData.className,
                                subject: currentSessionData.subject,
                                present: presentCount,
                                absent: absentCount,
                                total: studentsArr.length
                            });
                            saveLocalState();
                            renderReportsList();
                        }

                        try {
                            await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`, { method: 'DELETE' });
                        } catch (e) { console.error(e); }

                        currentSessionId = null;
                        currentSessionData = null;

                        btn.className = "w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-3 rounded-xl shadow-md transition duration-200 flex items-center justify-center space-x-2 text-sm";
                        btn.innerHTML = `<i class="fa-solid fa-play"></i><span>🟢 เริ่มเปิดคาบเรียน (Cloud Sync)</span>`;
                        classSelect.disabled = false;
                        subjectInput.disabled = false;
                        document.getElementById('active-session-container').classList.add('hidden');

                        Swal.fire('ปิดคาบเรียนเรียบร้อย', 'บันทึกรายงานประวัติสำเร็จแล้ว', 'success');
                    }
                });
            }
        }

        async function fetchTeacherSessionData() {
            if (!currentSessionId) return;
            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`);
                const data = await res.json();
                if (data) {
                    currentSessionData = data;
                    renderTeacherLiveTable(data);
                }
            } catch (e) { console.error('Cloud Poll Error:', e); }
        }

        function renderTeacherLiveTable(sessionData) {
            const tbody = document.getElementById('live-students-tbody');
            if (!sessionData || !sessionData.students) return;

            const studentsList = Object.values(sessionData.students).sort((a,b) => a.no - b.no);
            
            let presentCount = 0;
            let pendingCount = 0;
            let absentCount = 0;

            tbody.innerHTML = studentsList.map(s => {
                let statusBadge = '';
                let actionBtn = '';

                if (s.status === 'present') {
                    presentCount++;
                    statusBadge = `<span class="bg-emerald-100 text-emerald-800 text-xs px-2.5 py-1 rounded-full font-bold border border-emerald-200"><i class="fa-solid fa-check mr-1"></i>เข้าเรียน</span>`;
                    actionBtn = `<button onclick="updateStudentStatus('${s.id}', 'absent')" class="text-xs text-slate-400 hover:text-red-600 font-medium">ยกเลิก</button>`;
                } else if (s.status === 'pending') {
                    pendingCount++;
                    statusBadge = `<span class="bg-amber-100 text-amber-800 text-xs px-2.5 py-1 rounded-full font-bold border border-amber-200 pulse-slow"><i class="fa-solid fa-clock mr-1"></i>รออนุมัติ</span>`;
                    actionBtn = `<button onclick="approveStudent('${s.id}')" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs px-3 py-1 rounded-lg font-semibold shadow transition active:scale-95">อนุมัติ</button>`;
                } else {
                    absentCount++;
                    statusBadge = `<span class="bg-slate-100 text-slate-500 text-xs px-2.5 py-1 rounded-full font-medium">ยังไม่เช็ค</span>`;
                    actionBtn = `<button onclick="updateStudentStatus('${s.id}', 'present')" class="bg-blue-50 hover:bg-blue-100 text-blue-600 text-xs px-2.5 py-1 rounded-lg font-semibold border border-blue-200 transition">เช็คเข้าเรียน</button>`;
                }

                return `
                    <tr class="hover:bg-slate-50/80 transition">
                        <td class="p-3.5 font-semibold text-slate-700">${s.no}</td>
                        <td class="p-3.5 font-mono text-xs text-slate-500">${s.stdId}</td>
                        <td class="p-3.5 font-medium text-slate-800">${s.name}</td>
                        <td class="p-3.5 text-center">${statusBadge}</td>
                        <td class="p-3.5 text-center font-mono font-bold text-amber-600 tracking-wider text-base">${s.otp || '-'}</td>
                        <td class="p-3.5 text-center">${actionBtn}</td>
                    </tr>
                `;
            }).join('');

            document.getElementById('cnt-present').textContent = presentCount;
            document.getElementById('cnt-pending').textContent = pendingCount;
            document.getElementById('cnt-absent').textContent = absentCount;
        }

        async function approveStudent(stdId) {
            if (!currentSessionId) return;
            try {
                await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}/students/${stdId}.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ status: 'present' })
                });
                fetchTeacherSessionData();
            } catch (e) { console.error(e); }
        }

        async function updateStudentStatus(stdId, status) {
            if (!currentSessionId) return;
            try {
                await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}/students/${stdId}.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ status: status, otp: '' })
                });
                fetchTeacherSessionData();
            } catch (e) { console.error(e); }
        }

        async function verifyTeacherOTP() {
            const otpInput = document.getElementById('teacher-otp-input');
            const otpVal = otpInput.value.trim();
            if (!otpVal || otpVal.length !== 6) {
                Swal.fire({ icon: 'warning', title: 'รหัส OTP ไม่ถูกต้อง', text: 'กรุณากรอกรหัส OTP ให้ครบ 6 หลัก' });
                return;
            }

            if (!currentSessionData || !currentSessionData.students) return;

            const match = Object.values(currentSessionData.students).find(s => s.otp === otpVal && s.status === 'pending');
            if (match) {
                await approveStudent(match.id);
                otpInput.value = '';
                Swal.fire({ icon: 'success', title: 'ยืนยันสำเร็จ!', text: `เช็คชื่อให้ ${match.name} เรียบร้อยแล้ว`, timer: 1500, showConfirmButton: false });
            } else {
                Swal.fire({ icon: 'error', title: 'ไม่พบรหัส OTP นี้', text: 'รหัสไม่ถูกต้อง หรือนักเรียนอาจได้รับการอนุมัติไปแล้ว' });
            }
        }

        function copySessionLink() {
            const input = document.getElementById('session-link-input');
            input.select();
            navigator.clipboard.writeText(input.value);
            Swal.fire({ icon: 'success', title: 'คัดลอกลิงก์สำเร็จ', text: 'สามารถส่งลิงก์ให้กลุ่มนักเรียนสแกนเช็คชื่อได้ทันที', timer: 1500, showConfirmButton: false });
        }

        // Robust Student Fetching Logic
        async function fetchStudentSessionData(manualRefresh = false) {
            if (!currentSessionId) {
                currentSessionId = getSessionIdFromUrl();
            }
            
            if (!currentSessionId) {
                document.getElementById('student-class-title').textContent = "ไม่พบข้อมูลคาบเรียน";
                document.getElementById('student-session-info').textContent = "ลิงก์เช็คชื่อไม่ถูกต้อง หรือคาบเรียนปิดใช้งานแล้ว";
                document.getElementById('student-step-select').classList.add('hidden');
                return;
            }

            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}.json`);
                const data = await res.json();

                if (!data || data === null || !data.active) {
                    document.getElementById('student-class-title').textContent = "คาบเรียนนี้ถูกปิดแล้ว";
                    document.getElementById('student-session-info').textContent = "คุณครูได้ทำการปิดคาบเรียนนี้แล้ว ไม่สามารถเช็คชื่อเพิ่มได้";
                    document.getElementById('student-step-select').classList.add('hidden');
                    return;
                }

                // Activate Student UI
                document.getElementById('student-step-select').classList.remove('hidden');
                document.getElementById('student-class-title').textContent = `ห้อง ${data.className}`;
                document.getElementById('student-session-info').textContent = `วิชา: ${data.subject}`;

                const dropdown = document.getElementById('student-dropdown');
                const previousVal = dropdown.value;
                const studentsList = Object.values(data.students || {}).sort((a,b) => a.no - b.no);

                dropdown.innerHTML = '<option value="">-- เลือกชื่อของคุณ --</option>' + 
                    studentsList.map(s => `<option value="${s.id}">${s.no}. ${s.name} (${s.stdId})</option>`).join('');
                
                if (previousVal) dropdown.value = previousVal;

                if (currentStudentSelectedId && data.students[currentStudentSelectedId]) {
                    const myState = data.students[currentStudentSelectedId];
                    if (myState.status === 'present') {
                        document.getElementById('student-step-select').classList.add('hidden');
                        document.getElementById('student-step-otp').classList.add('hidden');
                        document.getElementById('student-step-success').classList.remove('hidden');
                    }
                }

                if (manualRefresh) {
                    Swal.fire({ icon: 'success', title: 'อัปเดตข้อมูลสำเร็จ', timer: 1000, showConfirmButton: false });
                }

            } catch (e) {
                document.getElementById('student-session-info').textContent = "เกิดข้อผิดพลาดในการเชื่อมต่อ Cloud Database";
            }
        }

        async function requestStudentOTP() {
            const dropdown = document.getElementById('student-dropdown');
            const stdId = dropdown.value;
            if (!stdId) {
                Swal.fire({ icon: 'warning', title: 'โปรดเลือกชื่อของคุณ', text: 'กรุณาเลือกชื่อ-นามสกุลจากรายการก่อนกดรับรหัส OTP' });
                return;
            }

            currentStudentSelectedId = stdId;
            const generatedOTP = Math.floor(100000 + Math.random() * 900000).toString();

            try {
                await fetch(`${CLOUD_DB_BASE_URL}/activeSessions/${currentSessionId}/students/${stdId}.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        status: 'pending',
                        otp: generatedOTP
                    })
                });

                document.getElementById('student-step-select').classList.add('hidden');
                document.getElementById('student-step-otp').classList.remove('hidden');
                document.getElementById('display-otp-code').textContent = generatedOTP;

            } catch (e) {
                Swal.fire({ icon: 'error', title: 'เกิดข้อผิดพลาด', text: 'ไม่สามารถขอรับ OTP ได้ กรุณาลองใหม่อีกครั้ง' });
            }
        }

        function addClass(e) {
            e.preventDefault();
            const name = document.getElementById('input-class-name').value.trim();
            const desc = document.getElementById('input-class-desc').value.trim();

            if (!name) return;

            const newClass = { id: 'c_' + Date.now(), name, desc };
            window.appState.classes.push(newClass);
            saveLocalState();
            populateClassDropdowns();
            renderClassesList();

            document.getElementById('form-add-class').reset();
            Swal.fire({ icon: 'success', title: 'เพิ่มห้องเรียนสำเร็จ', timer: 1200, showConfirmButton: false });
        }

        function deleteClass(id) {
            Swal.fire({
                title: 'ยืนยันลบห้องเรียน?',
                text: "ข้อมูลนักเรียนทั้งหมดในห้องนี้จะถูกลบไปด้วย",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#ef4444',
                confirmButtonText: 'ลบห้องเรียน'
            }).then(res => {
                if (res.isConfirmed) {
                    window.appState.classes = window.appState.classes.filter(c => c.id !== id);
                    window.appState.students = window.appState.students.filter(s => s.classId !== id);
                    saveLocalState();
                    populateClassDropdowns();
                    renderClassesList();
                    renderStudentsList();
                }
            });
        }

        function renderClassesList() {
            const tbody = document.getElementById('classes-table-tbody');
            if (!tbody) return;

            if (window.appState.classes.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="text-center py-6 text-slate-400">ยังไม่มีข้อมูลห้องเรียน</td></tr>`;
                return;
            }

            tbody.innerHTML = window.appState.classes.map((c, idx) => {
                const count = window.appState.students.filter(s => s.classId === c.id).length;
                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3.5 font-semibold text-slate-600">${idx + 1}</td>
                        <td class="p-3.5 font-bold text-slate-800">${c.name}</td>
                        <td class="p-3.5 text-slate-500">${c.desc || '-'}</td>
                        <td class="p-3.5 text-center"><span class="bg-blue-50 text-blue-700 px-2.5 py-1 rounded-full font-bold text-xs">${count} คน</span></td>
                        <td class="p-3.5 text-center">
                            <button onclick="deleteClass('${c.id}')" class="text-rose-600 hover:text-rose-800 font-semibold text-xs"><i class="fa-solid fa-trash mr-1"></i>ลบ</button>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function addSingleStudent(e) {
            e.preventDefault();
            const classId = document.getElementById('single-std-class').value;
            const no = parseInt(document.getElementById('single-std-no').value);
            const stdId = document.getElementById('single-std-id').value.trim();
            const name = document.getElementById('single-std-name').value.trim();

            if (!classId || !no || !stdId || !name) return;

            window.appState.students.push({ id: 's_' + Date.now(), classId, no, stdId, name });
            saveLocalState();
            renderClassesList();
            renderStudentsList();

            e.target.reset();
            Swal.fire({ icon: 'success', title: 'เพิ่มนักเรียนสำเร็จ', timer: 1200, showConfirmButton: false });
        }

        function addBulkStudents(e) {
            e.preventDefault();
            const classId = document.getElementById('bulk-std-class').value;
            const text = document.getElementById('bulk-std-text').value.trim();

            if (!classId || !text) return;

            const lines = text.split('\n');
            let addedCount = 0;

            lines.forEach((line, idx) => {
                const parts = line.split('\t').map(p => p.trim());
                if (parts.length >= 3) {
                    const no = parseInt(parts[0]) || (idx + 1);
                    const stdId = parts[1];
                    const name = parts[2];
                    window.appState.students.push({ id: 's_' + Date.now() + '_' + idx, classId, no, stdId, name });
                    addedCount++;
                }
            });

            saveLocalState();
            renderClassesList();
            renderStudentsList();

            e.target.reset();
            Swal.fire({ icon: 'success', title: `นำเข้าสำเร็จ ${addedCount} รายการ`, timer: 1500, showConfirmButton: false });
        }

        function deleteStudent(id) {
            window.appState.students = window.appState.students.filter(s => s.id !== id);
            saveLocalState();
            renderClassesList();
            renderStudentsList();
        }

        function renderStudentsList() {
            const tbody = document.getElementById('students-table-tbody');
            const filterClassId = document.getElementById('filter-std-class').value;
            if (!tbody) return;

            let filtered = window.appState.students;
            if (filterClassId && filterClassId !== 'ALL') {
                filtered = filtered.filter(s => s.classId === filterClassId);
            }

            filtered.sort((a,b) => a.no - b.no);

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="text-center py-6 text-slate-400">ยังไม่มีรายชื่อนักเรียน</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(s => {
                const cObj = window.appState.classes.find(c => c.id === s.classId);
                return `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3.5"><span class="bg-slate-100 text-slate-700 text-xs px-2.5 py-1 rounded-md font-semibold">${cObj ? cObj.name : '-'}</span></td>
                        <td class="p-3.5 font-bold text-slate-700">${s.no}</td>
                        <td class="p-3.5 font-mono text-xs text-slate-500">${s.stdId}</td>
                        <td class="p-3.5 font-medium text-slate-800">${s.name}</td>
                        <td class="p-3.5 text-center">
                            <button onclick="deleteStudent('${s.id}')" class="text-rose-600 hover:text-rose-800 font-semibold text-xs"><i class="fa-solid fa-trash mr-1"></i>ลบ</button>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function renderReportsList() {
            const tbody = document.getElementById('reports-table-tbody');
            if (!tbody) return;

            if (window.appState.history.length === 0) {
                tbody.innerHTML = `<tr><td colspan="6" class="text-center py-6 text-slate-400">ยังไม่มีประวัติการเช็คชื่อ</td></tr>`;
                return;
            }

            tbody.innerHTML = window.appState.history.map(h => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3.5 font-mono text-xs text-slate-500">${h.timestamp}</td>
                    <td class="p-3.5 font-bold text-slate-800">${h.className}</td>
                    <td class="p-3.5 text-slate-600">${h.subject}</td>
                    <td class="p-3.5 text-center"><span class="bg-emerald-50 text-emerald-700 font-bold px-2.5 py-1 rounded-full text-xs">${h.present} คน</span></td>
                    <td class="p-3.5 text-center"><span class="bg-rose-50 text-rose-700 font-bold px-2.5 py-1 rounded-full text-xs">${h.absent} คน</span></td>
                    <td class="p-3.5 text-center"><span class="bg-slate-100 text-slate-600 font-semibold px-2.5 py-1 rounded-full text-xs">เสร็จสิ้น</span></td>
                </tr>
            `).join('');
        }

        function exportReportsToCSV() {
            if (window.appState.history.length === 0) {
                Swal.fire({ icon: 'info', title: 'ไม่มีข้อมูลรายงาน', text: 'ยังไม่มีข้อมูลประวัติการเช็คชื่อสำหรับส่งออก' });
                return;
            }

            let csv = '\uFEFFวัน-เวลา,ห้องเรียน,วิชา,มาเรียน,ขาดเรียน,จำนวนทั้งหมด\n';
            window.appState.history.forEach(h => {
                csv += `"${h.timestamp}","${h.className}","${h.subject}",${h.present},${h.absent},${h.total}\n`;
            });

            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.setAttribute("download", `CheckIn_Report_${Date.now()}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }
    </script>
</body>
</html>
