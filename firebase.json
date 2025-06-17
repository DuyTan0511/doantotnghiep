// Biến global
let websocket = null;
let reconnectInterval = null;

// Cấu hình Firebase - THAY BẰNG CONFIG CỦA BẠN
const firebaseConfig = {
  apiKey: "AIzaSyBceqX7RLapoQoDxVoAFTz-cG6UvZyNhZs",
  authDomain: "esp32-duan.firebaseapp.com",
  databaseURL: "https://esp32-duan-default-rtdb.firebaseio.com",
  projectId: "esp32-duan",
  storageBucket: "esp32-duan.firebasestorage.app",
  messagingSenderId: "119386388188",
  appId: "1:119386388188:web:111bc15cba677d12092619",
  measurementId: "G-6T8ZG83NZR"
};

// Khởi tạo Firebase
function initFirebase() {
    try {
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
        
        // Lắng nghe dữ liệu flowRate từ Firebase
        db.ref("flowRate").on("value", (snapshot) => {
            const value = snapshot.val();
            if (value !== null) {
                document.getElementById("flowValue").innerText = value.toFixed(2);
                updateFirebaseStatus(true);
                addLog("📊 Firebase: " + value.toFixed(2) + " L/min");
            }
        });
        
        // Kiểm tra trạng thái kết nối Firebase
        db.ref(".info/connected").on("value", (snapshot) => {
            const connected = snapshot.val();
            updateFirebaseStatus(connected);
            if (connected) {
                addLog("✅ Firebase đã kết nối");
            } else {
                addLog("❌ Firebase mất kết nối");
            }
        });
        
    } catch (error) {
        addLog("❌ Lỗi Firebase: " + error.message);
        updateFirebaseStatus(false);
    }
}

// Hàm kết nối WebSocket tới ESP32
function connectWebSocket() {
    const ip = document.getElementById('espIP').value.trim();
    if (!ip) {
        addLog("⚠️ Vui lòng nhập IP ESP32");
        return;
    }
    
    if (websocket && websocket.readyState === WebSocket.OPEN) {
        addLog("ℹ️ WebSocket đã kết nối");
        return;
    }
    
    const wsUrl = `ws://${ip}/ws`;
    addLog("🔄 Đang kết nối tới: " + wsUrl);
    
    try {
        websocket = new WebSocket(wsUrl);
        
        websocket.onopen = function(event) {
            addLog("✅ WebSocket kết nối thành công!");
            updateWebSocketStatus(true);
            clearInterval(reconnectInterval);
        };
        
        websocket.onmessage = function(event) {
            addLog("📩 ESP32: " + event.data);
            
            // Nếu nhận được dữ liệu số, cập nhật hiển thị
            const flowRate = parseFloat(event.data);
            if (!isNaN(flowRate)) {
                document.getElementById("flowValue").innerText = flowRate.toFixed(2);
            }
        };
        
        websocket.onclose = function(event) {
            addLog("❌ WebSocket đã đóng (Code: " + event.code + ")");
            updateWebSocketStatus(false);
            
            // Tự động kết nối lại sau 5 giây
            reconnectInterval = setInterval(() => {
                addLog("🔄 Đang thử kết nối lại...");
                connectWebSocket();
            }, 5000);
        };
        
        websocket.onerror = function(error) {
            addLog("❌ Lỗi WebSocket");
            updateWebSocketStatus(false);
        };
        
    } catch (error) {
        addLog("❌ Không thể tạo WebSocket: " + error.message);
        updateWebSocketStatus(false);
    }
}

// Hàm ngắt kết nối WebSocket
function disconnectWebSocket() {
    if (websocket) {
        websocket.close();
        clearInterval(reconnectInterval);
        addLog("🔌 Đã ngắt kết nối WebSocket");
    }
}

// Hàm gửi tin nhắn test
function sendTestMessage() {
    if (websocket && websocket.readyState === WebSocket.OPEN) {
        const message = "Test from web: " + new Date().toLocaleTimeString();
        websocket.send(message);
        addLog("📤 Đã gửi: " + message);
    } else {
        addLog("⚠️ WebSocket chưa kết nối");
    }
}

// Hàm cập nhật trạng thái WebSocket UI
function updateWebSocketStatus(connected) {
    const indicator = document.getElementById('wsStatus');
    const text = document.getElementById('wsStatusText');
    
    if (connected) {
        indicator.className = 'status-indicator connected';
        text.textContent = 'Đã kết nối';
    } else {
        indicator.className = 'status-indicator disconnected';
        text.textContent = 'Mất kết nối';
    }
}

// Hàm cập nhật trạng thái Firebase UI
function updateFirebaseStatus(connected) {
    const indicator = document.getElementById('fbStatus');
    const text = document.getElementById('fbStatusText');
    
    if (connected) {
        indicator.className = 'status-indicator connected';
        text.textContent = 'Đã kết nối';
    } else {
        indicator.className = 'status-indicator disconnected';
        text.textContent = 'Mất kết nối';
    }
}

// Hàm thêm log vào UI
function addLog(message) {
    const logContainer = document.getElementById('logContainer');
    const timestamp = new Date().toLocaleTimeString();
    const logEntry = document.createElement('div');
    logEntry.className = 'log-entry';
    logEntry.innerHTML = `<span class="timestamp">[${timestamp}]</span>${message}`;
    
    logContainer.appendChild(logEntry);
    logContainer.scrollTop = logContainer.scrollHeight;
    
    // Giới hạn số dòng log (tối đa 100)
    if (logContainer.children.length > 100) {
        logContainer.removeChild(logContainer.firstChild);
    }
}

// Hàm xóa log
function clearLog() {
    document.getElementById('logContainer').innerHTML = '';
    addLog("🗑️ Log đã được xóa");
}

// Khởi tạo khi trang load
window.onload = function() {
    addLog("🚀 Hệ thống khởi động");
    addLog("ℹ️ Nhập IP ESP32 và nhấn 'Kết nối ESP32'");
    
    // Khởi tạo Firebase
    initFirebase();
};

// Dọn dẹp khi đóng trang  
window.onbeforeunload = function() {
    if (websocket) {
        websocket.close();
    }
};
