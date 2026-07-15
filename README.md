<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Anemia Screening - ระบบคัดกรองภาวะซีด</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Sarabun', sans-serif;
        }
        body {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .app-card {
            background: white;
            width: 100%;
            max-width: 450px;
            border-radius: 24px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            padding: 30px;
            text-align: center;
        }
        .header h1 {
            font-size: 22px;
            color: #2c3e50;
            margin-bottom: 5px;
        }
        .header p {
            font-size: 14px;
            color: #7f8c8d;
            margin-top: 0;
        }
        .camera-box {
            width: 100%;
            aspect-ratio: 1 / 1;
            background-color: #222;
            border-radius: 20px;
            margin: 25px 0;
            position: relative;
            overflow: hidden;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.6);
            display: flex;
            justify-content: center;
            align-items: center;
        }
        /* กรอบเล็งเป้าสำหรับตรวจเปลือกตา */
        .camera-overlay {
            position: absolute;
            width: 70%;
            height: 35%;
            border: 3px solid #00eca6;
            border-radius: 50% / 20%;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.5);
            pointer-events: none;
            z-index: 10;
        }
        .camera-overlay::after {
            content: "จัดวางเยื่อบุตาให้อยู่ในกรอบนี้";
            position: absolute;
            bottom: -30px;
            left: 50%;
            transform: translateX(-50%);
            color: #00eca6;
            font-size: 12px;
            white-space: nowrap;
            font-weight: bold;
            text-shadow: 1px 1px 3px black;
        }
        #webcam-container canvas {
            width: 100% !important;
            height: 100% !important;
            object-fit: cover;
        }
        .btn-status {
            font-size: 14px;
            color: #95a5a6;
            margin-bottom: 15px;
        }
        .btn-action {
            background: linear-gradient(135deg, #007bff 0%, #0056b3 100%);
            color: white;
            border: none;
            width: 100%;
            padding: 15px;
            font-size: 18px;
            font-weight: 600;
            border-radius: 15px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 5px 15px rgba(0,123,255,0.3);
        }
        .btn-action:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(0,123,255,0.4);
        }
        .result-section {
            margin-top: 25px;
            padding: 20px;
            border-radius: 15px;
            background-color: #f8f9fa;
            border: 1px dashed #e2e8f0;
            display: none;
        }
        .result-title {
            font-size: 14px;
            color: #7f8c8d;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        .result-value {
            font-size: 24px;
            font-weight: 700;
            color: #e74c3c;
            margin: 10px 0;
        }
        .result-tip {
            font-size: 13px;
            color: #7f8c8d;
            line-height: 1.5;
        }
        .placeholder-text {
            color: #fff;
            font-size: 14px;
            z-index: 1;
        }
    </style>
</head>
<body>

<div class="app-card">
    <div class="header">
        <h1>EyeScreen AI 👁️</h1>
        <p>ระบบคัดกรองภาวะซีดจากเยื่อบุตาเบื้องต้นด้วย AI</p>
    </div>

    <div class="camera-box">
        <div class="camera-overlay" id="overlay" style="display: none;"></div>
        <div id="webcam-container">
            <span class="placeholder-text" id="placeholder">กดปุ่มด้านล่างเพื่อเปิดใช้งานกล้อง</span>
        </div>
    </div>

    <div class="btn-status" id="status-text">ระบบพร้อมทำงาน</div>
    <button class="btn-action" id="main-btn" onclick="startApp()">เปิดกล้องและเริ่มวิเคราะห์</button>

    <div class="result-section" id="result-box">
        <div class="result-title">ผลการวิเคราะห์เบื้องต้น</div>
        <div class="result-value" id="result-value">กำลังประมวลผล...</div>
        <div class="result-tip">
            *หมายเหตุ: ระบบนี้เป็นเพียงการคัดกรองเบื้องต้นเท่านั้น ไม่สามารถใช้ทดแทนการเจาะเลือดและวินิจฉัยโดยแพทย์ได้
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

<script>
    // 🔴 ส่วนสำคัญ: เมื่อเทรน AI จากเว็บ Teachable Machine เสร็จแล้ว ให้เอาลิงก์มาวางแทนที่ตรงนี้ครับ
    const URL = "https://teachablemachine.withgoogle.com/models/YOUR_MODEL_ID/";

    let model, webcam, ctx, maxPredictions;
    let isRunning = false;

    async function startApp() {
        if (isRunning) return;
        
        const btn = document.getElementById("main-btn");
        const statusText = document.getElementById("status-text");
        const placeholder = document.getElementById("placeholder");
        const overlay = document.getElementById("overlay");
        const resultBox = document.getElementById("result-box");

        btn.disabled = true;
        statusText.innerHTML = "กำลังโหลดโมเดล AI... โปรดรอสักครู่";
        
        try {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            // โหลดโมเดล AI
            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

            statusText.innerHTML = "กำลังเปิดกล้องหลัง...";
            
            // ตั้งค่ากล้อง (บังคับเปิดกล้องหลังมือถือ)
            const flip = false; 
            webcam = new tmImage.Webcam(400, 400, flip); 
            await webcam.setup({ facingMode: "environment" }); 
            await webcam.play();
            
            // ซ่อนข้อความเริ่มต้น และแสดงกรอบเล็ง
            placeholder.style.display = "none";
            overlay.style.display = "block";
            resultBox.style.display = "block";
            
            // นำวิดีโอเข้าหน้าจอ
            document.getElementById("webcam-container").appendChild(webcam.canvas);
            
            isRunning = true;
            statusText.innerHTML = "🔴 กำลังวิเคราะห์แบบ Real-time";
            btn.innerHTML = "ระบบกำลังทำงาน...";
            btn.style.background = "#95a5a6";
            
            window.requestAnimationFrame(loop);
        } catch (error) {
            alert("ไม่สามารถเข้าถึงกล้องหรือโหลดโมเดลได้: " + error);
            statusText.innerHTML = "เกิดข้อผิดพลาด";
            btn.disabled = false;
        }
    }

    async function loop() {
        if (!isRunning) return;
        webcam.update(); 
        await predict();
        window.requestAnimationFrame(loop);
    }

    async function predict() {
        const prediction = await model.predict(webcam.canvas);
        
        // ค้นหาคลาสที่ AI มั่นใจที่สุด
        let highestPred = prediction.reduce((prev, current) => (prev.probability > current.probability) ? prev : current);
        
        const resultValue = document.getElementById("result-value");
        
        // ปรับการแสดงผลตามผลลัพธ์
        let classNameTH = highestPred.className;
        if(classNameTH === "Anemia" || classNameTH === "Pale") {
            classNameTH = "⚠️ มีภาวะซีด";
            resultValue.style.color = "#e74c3c";
        } else if(classNameTH === "Normal") {
            classNameTH = "✅ ปกติ";
            resultValue.style.color = "#2ecc71";
        }

        resultValue.innerHTML = `${classNameTH} <span style="font-size:16px; font-weight:normal; color:#7f8c8d;">(${ (highestPred.probability * 100).toFixed(0) }%)</span>`;
    }
</script>
</body>
</html>
