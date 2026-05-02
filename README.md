<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Object Vault Pro</title>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
    <style>
        :root { --accent: #00f2fe; --bg: #0a0a0a; }
        body { background: var(--bg); color: white; font-family: 'Segoe UI', sans-serif; margin: 0; text-align: center; }
        
        #lock-screen { position: fixed; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; background: var(--bg); padding: 20px; }
        
        /* Video Container */
        .video-box { position: relative; width: 300px; height: 300px; border-radius: 50%; border: 4px solid var(--accent); overflow: hidden; box-shadow: 0 0 30px rgba(0,242,254,0.2); margin-bottom: 20px; }
        #video { width: 100%; height: 100%; object-fit: cover; }

        .btn-start { background: var(--accent); color: black; border: none; padding: 15px 40px; border-radius: 50px; font-weight: bold; cursor: pointer; font-size: 1.1rem; }
        
        #gallery { display: none; padding: 15px; grid-template-columns: repeat(3, 1fr); gap: 8px; }
        .photo-card { width: 100%; aspect-ratio: 1; background: #222; border-radius: 4px; object-fit: cover; }
        
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="lock-screen">
        <h1 style="letter-spacing: 4px;">SECURE VAULT</h1>
        <div class="video-box">
            <video id="video" autoplay playsinline muted></video>
        </div>
        <div id="status" style="margin-bottom: 20px; color: #888;">AI System Offline</div>
        <button id="start-btn" class="btn-start" onclick="initSystem()">Initialize Scanner</button>
    </div>

    <div id="gallery-container" class="hidden">
        <h2 style="padding: 20px; border-bottom: 1px solid #333;">Private Photos</h2>
        <div id="gallery">
            <img class="photo-card" src="https://picsum.photos/300/300?1">
            <img class="photo-card" src="https://picsum.photos/300/300?2">
            <img class="photo-card" src="https://picsum.photos/300/300?3">
            <img class="photo-card" src="https://picsum.photos/300/300?4">
            <img class="photo-card" src="https://picsum.photos/300/300?5">
            <img class="photo-card" src="https://picsum.photos/300/300?6">
        </div>
    </div>

    <script>
        const video = document.getElementById('video');
        const status = document.getElementById('status');
        const startBtn = document.getElementById('start-btn');
        
        // KEY OBJECT: Change to 'bottle', 'cup', 'remote', etc.
        const KEY_OBJECT = "cell phone"; 

        async function initSystem() {
            startBtn.classList.add('hidden');
            status.innerText = "Loading AI Model...";
            
            try {
                // 1. Load Model
                const model = await cocossd.load();
                status.innerText = "Camera Accessing...";

                // 2. Start Camera
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user" } 
                });
                video.srcObject = stream;

                status.innerText = `Scanning for: ${KEY_OBJECT.toUpperCase()}`;
                
                // 3. Detection Loop
                const checkFrame = async () => {
                    const predictions = await model.detect(video);
                    const found = predictions.find(p => p.class === KEY_OBJECT && p.score > 0.60);

                    if (found) {
                        status.innerText = "Access Granted!";
                        status.style.color = "#00ff00";
                        setTimeout(unlock, 1000);
                    } else {
                        requestAnimationFrame(checkFrame);
                    }
                };
                
                checkFrame();

            } catch (err) {
                status.innerText = "Error: " + err.message;
                startBtn.classList.remove('hidden');
            }
        }

        function unlock() {
            document.getElementById('lock-screen').classList.add('hidden');
            document.getElementById('gallery-container').classList.remove('hidden');
            document.getElementById('gallery').style.display = 'grid';
            
            // Stop Camera
            if (video.srcObject) {
                video.srcObject.getTracks().forEach(track => track.stop());
            }
        }
    </script>
</body>
</html>
