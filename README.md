<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إضافة سكربت جديد</title>
<style>
        body {
            font-family: 'Cairo', sans-serif;
            background-color: #f8f9fa;
            color: #343a40;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            padding: 0;
        }
        .form-container {
            background-color: #fff;
            border-radius: 12px;
            padding: 40px;
            width: 100%;
            max-width: 500px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
            text-align: center;
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .form-container:hover {
            transform: translateY(-5px);
            box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
        }
        .header h1 {
            font-size: 24px;
            color: #007bff;
            margin-bottom: 30px;
            border-bottom: 1px solid #ddd;
            padding-bottom: 10px;
        }
        .form-group {
            margin-bottom: 20px;
            text-align: left;
        }
        .form-group label {
            display: block;
            font-size: 16px;
            color: #495057;
            margin-bottom: 5px;
        }
        .form-group input, .form-group textarea {
            width: 100%;
            padding: 12px;
            font-size: 16px;
            color: #495057;
            border: 1px solid #ced4da;
            border-radius: 6px;
            transition: border-color 0.3s;
        }
        .form-group input:focus, .form-group textarea:focus {
            border-color: #007bff;
            outline: none;
        }
        .submit-button {
            padding: 12px 0;
            width: 100%;
            font-size: 18px;
            color: #fff;
            background-color: #007bff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s;
        }
        .submit-button:hover {
            background-color: #0056b3;
            transform: translateY(-2px);
        }
        .copy-message, .exists-message {
            display: none;
            color: green;
            font-weight: bold;
            margin-top: 15px;
        }
        .copy-message {
            color: #28a745;
        }
        .exists-message {
            color: #dc3545;
        }
        .exists-message a {
            color: #dc3545;
            text-decoration: underline;
        }
    </style>
</head>
<body>

<div class="header">
    <h1>إضافة سكربت جديد</h1>
</div>

<div class="form-container">
    <form id="addScriptForm">
        <div class="form-group">
            <label for="subscriptionCode">كود الاشتراك:</label>
            <input type="text" id="subscriptionCode" required>
        </div>
        <div class="form-group">
            <label for="title">عنوان السكربت:</label>
            <input type="text" id="title" required>
        </div>
        <div class="form-group">
            <label for="mapName">اسم اللعبة:</label>
            <input type="text" id="mapName" required>
        </div>
        <div class="form-group">
            <label for="description">الوصف:</label>
            <textarea id="description" rows="4" required></textarea>
        </div>
        <div class="form-group">
            <label for="scriptName">الاسكربت:</label>
            <input type="text" id="scriptName" required>
        </div>
        <div class="form-group">
            <label for="image">رفع صورة:</label>
            <input type="file" id="image" accept="image/*" required>
        </div>
        <button type="submit" class="submit-button">إضافة السكربت</button>
    </form>
    <div class="copy-message" id="copyMessage">تم نسخ رابط السكربت بنجاح!</div>
    <div class="exists-message" id="existsMessage"></div>
</div>

<script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-storage-compat.js"></script>

<script>
    const firebaseConfig = {
        apiKey: "AIzaSyDoK427q44Fj87JhgZh2vKeUnACkjl8HDc",
        authDomain: "treng-c6027.firebaseapp.com",
        projectId: "treng-c6027",
        storageBucket: "treng-c6027.appspot.com",
        messagingSenderId: "305527546716",
        appId: "1:305527546716:web:fd072a58200f31bb42d799",
        measurementId: "G-3JFKFT2LT0"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
    const storage = firebase.storage();

    document.getElementById("addScriptForm").addEventListener("submit", async (e) => {
        e.preventDefault();

        const subscriptionCode = document.getElementById("subscriptionCode").value;
        const title = document.getElementById("title").value;
        const mapName = document.getElementById("mapName").value;
        const description = document.getElementById("description").value;
        const scriptName = document.getElementById("scriptName").value;
        const imageFile = document.getElementById("image").files[0];
        const timestamp = new Date().toLocaleDateString();

        const isValidSubscription = await checkSubscription(subscriptionCode);
        if (!isValidSubscription) {
            alert("كود الاشتراك غير صالح أو انتهت صلاحيته.");
            return;
        }

        const existingScript = await checkIfScriptExists(scriptName);
        if (existingScript) {
            displayExistingScriptMessage(existingScript.id);
            return;
        }

        if (!imageFile) {
            alert("يرجى رفع صورة.");
            return;
        }

        try {
            const storageRef = storage.ref(`scripts/${imageFile.name}`);
            await storageRef.put(imageFile);
            const imageUrl = await storageRef.getDownloadURL();

            const docRef = await db.collection("scripts").add({
                title,
                mapName,
                description,
                scriptName,
                subscriptionCode,
                imageUrl,
                addedDate: timestamp
            });

            const scriptUrl = `https://h-scripts.site/view.html?id=${docRef.id}`;
            copyToClipboard(scriptUrl);
            showCopyMessage();

            document.getElementById("addScriptForm").reset();
        } catch (error) {
            console.error("حدث خطأ:", error);
            alert("حدث خطأ أثناء إضافة السكربت.");
        }
    });

    async function checkIfScriptExists(scriptName) {
        try {
            const scriptSnapshot = await db.collection("scripts").where("scriptName", "==", scriptName).get();
            return scriptSnapshot.empty ? null : scriptSnapshot.docs[0];
        } catch (error) {
            console.error("خطأ أثناء التحقق من وجود السكربت:", error);
            return null;
        }
    }

    async function checkSubscription(code) {
        try {
            const subscriptionDoc = await db.collection("subscriptions").where("subscriptionCode", "==", code).get();
            if (subscriptionDoc.empty) {
                return false;
            }

            const subscriptionData = subscriptionDoc.docs[0].data();
            const expiryDate = subscriptionData.expiresAt.toDate();
            const now = new Date();
            return expiryDate > now;
        } catch (error) {
            console.error("خطأ أثناء التحقق من الاشتراك:", error);
            return false;
        }
    }

    function copyToClipboard(text) {
        const tempInput = document.createElement("input");
        tempInput.value = text;
        document.body.appendChild(tempInput);
        tempInput.select();
        document.execCommand("copy");
        document.body.removeChild(tempInput);
    }

    function showCopyMessage() {
        const copyMessage = document.getElementById("copyMessage");
        copyMessage.style.display = "block";
        setTimeout(() => {
            copyMessage.style.display = "none";
        }, 3000);
    }

    function displayExistingScriptMessage(scriptId) {
        const existsMessage = document.getElementById("existsMessage");
        const scriptUrl = `https://h-scripts.site/view.html?id=${scriptId}`;
        existsMessage.innerHTML = `هذا السكربت موجود مسبقًا. <a href="${scriptUrl}" target="_blank">افتح السكربت هنا</a>`;
        existsMessage.style.display = "block";
    }
</script>


<script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-firestore-compat.js"></script>

<script>
    const myFirebaseConfig = {
        apiKey: "AIzaSyDoK427q44Fj87JhgZh2vKeUnACkjl8HDc",
        authDomain: "treng-c6027.firebaseapp.com",
        projectId: "treng-c6027",
        storageBucket: "treng-c6027.appspot.com",
        messagingSenderId: "305527546716",
        appId: "1:305527546716:web:fd072a58200f31bb42d799",
        measurementId: "G-3JFKFT2LT0"
    };

    firebase.initializeApp(myFirebaseConfig);
    const myDb = firebase.firestore();

    // دالة التحقق من حالة الصيانة
    async function checkMaintenanceStatus() {
        const doc = await myDb.collection("settings").doc("maintenance").get();
        if (doc.exists && doc.data().isActive) {
            document.body.innerHTML = `
                <div style="display: flex; flex-direction: column; align-items: center; text-align: center; margin-top: 10%; font-family: Arial, sans-serif;">
                    <img src="https://i.ibb.co/W077gdL/images-1-removebg-preview.png" alt="صورة الصيانة" style="max-width: 200px; border-radius: 10px; margin-bottom: 20px;">
                    <h1 style="font-size: 32px; color: #333; margin-bottom: 10px;">الموقع تحت الصيانة</h1>
                    <p style="font-size: 18px; color: #666; margin-bottom: 20px;">
                        نحن نعمل حاليًا على تحسين تجربة المستخدم. يرجى التحقق مرة أخرى لاحقًا.
                    </p>
                    <a href="https://t.me/hscripts" style="display: inline-block; padding: 12px 24px; background-color: #007bff; color: #fff; font-size: 18px; border-radius: 5px; text-decoration: none; transition: background-color 0.3s;">
                        تواصل مع الدعم
                    </a>
                </div>
            `;
        }
    }

    // التحقق من حالة الصيانة عند تحميل الصفحة
    checkMaintenanceStatus();
</script>

    

    
</body>
</html>
