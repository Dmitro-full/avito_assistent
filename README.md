# avito_assistent
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Утилиты</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: url('background.jpg') no-repeat center center fixed;
            background-size: cover;
            margin: 0;
        }
        .navbar {
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: flex-start;
        }
        .navbar a {
            color: white;
            margin: 0 15px;
            text-decoration: none;
            font-weight: bold;
            font-size: 18px;
        }
        .navbar a:hover {
            text-decoration: underline;
        }
        .container {
            background: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            width: 500px;
            margin: 60px auto 20px auto; 
            display: none;
        }
        .container.active {
            display: block;
        }
        textarea, input, button {
            width: 100%;
            margin-top: 10px;
            padding: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: 1px solid #ccc;
        }
        button {
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover {
            background-color: #218838;
        }
        .image-preview {
            display: flex;
            flex-wrap: wrap;
            margin-top: 20px;
            justify-content: center;
        }
        .image-preview img {
            margin: 5px;
            border: 2px solid #ccc;
            border-radius: 5px;
            max-width: 100%;
            max-height: 200px;
            object-fit: contain;
        }
        .processed-preview img {
            width: 720px;
            height: 480px;
            object-fit: contain;
            border: 2px solid #28a745;
        }
        .preview-container {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <!-- Навигационное меню -->
    <div class="navbar">
        <a href="javascript:void(0)" onclick="switchSection('dateSection')">Преобразование дат</a>
        <a href="javascript:void(0)" onclick="switchSection('photoSection')">Обработка фотографий</a>
        <a href="javascript:void(0)" onclick="switchSection('textGenerationSection')">Генерация текста</a> <!-- Новая ссылка -->
    </div>

    <!-- Страница для преобразования дат -->
    <div id="dateSection" class="container active">
        <h2>Преобразование дат</h2>
        <textarea id="dateInput" rows="5" placeholder="Введите даты в формате DD.MM.YYYY HH:MM..."></textarea>
        <input type="number" id="shiftMinutes" placeholder="Минуты для сдвига">
        <button onclick="shiftDates()">Преобразовать</button>
        <h3>Результат:</h3>
        <textarea id="output" rows="5" readonly></textarea>
        <button onclick="copyToClipboard()">Скопировать</button>
    </div>

    <!-- Страница для обработки фотографий -->
    <div id="photoSection" class="container">
        <h2>Пакетная обработка фотографий</h2>
        <input type="file" id="imageInput" multiple accept="image/*" onchange="previewImages()">
        <button onclick="processImages()">Обработать фотографии</button>

        <div class="preview-container">
            <h3>Предпросмотр исходных фото:</h3>
            <div id="originalPreview" class="image-preview"></div>

            <h3>Предпросмотр обработанных фото:</h3>
            <div id="processedPreview" class="image-preview processed-preview"></div>
        </div>

        <canvas id="canvas" style="display:none;"></canvas>

        <button id="downloadBtn" style="display:none;" onclick="downloadImages()">Скачать обработанные фото</button>
    </div>

    <!-- Страница для генерации текста -->
    <div id="textGenerationSection" class="container">
        <h2>Генерация текста</h2>
        
        <textarea id="condition1" rows="3" placeholder="Условия 1"></textarea>
        <textarea id="condition2" rows="3" placeholder="Условия 2"></textarea>
        <textarea id="condition3" rows="3" placeholder="Условия 3"></textarea>
        <textarea id="condition4" rows="3" placeholder="Условия 4"></textarea>
        <textarea id="condition5" rows="3" placeholder="Условия 5"></textarea>

        <button onclick="generateText()">Генерировать текст</button>
        
        <h3>Результат генерации:</h3>
        <textarea id="generatedText" rows="5" readonly></textarea>

        <button onclick="copyToClipboard()">Скопировать результат</button>
    </div>

    <script>
        // Функция для переключения между разделами
        function switchSection(sectionId) {
            let containers = document.querySelectorAll('.container');
            containers.forEach(container => {
                container.classList.remove('active');
            });
            document.getElementById(sectionId).classList.add('active');
        }

        // Функции для преобразования дат
        function parseDate(dateStr) {
            let parts = dateStr.match(/(\d{2})\.(\d{2})\.(\d{4}) (\d{2}):(\d{2})/);
            if (parts) {
                return new Date(parts[3], parts[2] - 1, parts[1], parts[4], parts[5]);
            }
            return NaN;
        }

        function shiftDates() {
            let input = document.getElementById("dateInput").value.trim().split("\n");
            let shiftMinutes = parseInt(document.getElementById("shiftMinutes").value);
            let output = [];
            
            input.forEach(dateStr => {
                let date = parseDate(dateStr);
                if (!isNaN(date)) {
                    date.setMinutes(date.getMinutes() + shiftMinutes);
                    let formattedDate = `${("0" + date.getDate()).slice(-2)}.${("0" + (date.getMonth() + 1)).slice(-2)}.${date.getFullYear()} ${("0" + date.getHours()).slice(-2)}:${("0" + date.getMinutes()).slice(-2)}`;
                    output.push(formattedDate);
                } else {
                    output.push("Неверный формат: " + dateStr);
                }
            });
            
            document.getElementById("output").value = output.join("\n");
        }

        function copyToClipboard() {
            let outputField = document.getElementById("output");
            outputField.select();
            outputField.setSelectionRange(0, 99999);
            document.execCommand("copy");
        }

        // Функции для обработки фотографий
        let processedImages = [];

        function previewImages() {
            const files = document.getElementById('imageInput').files;
            const originalPreview = document.getElementById('originalPreview');
            originalPreview.innerHTML = '';

            Array.from(files).forEach(file => {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = new Image();
                    img.onload = function() {
                        const originalImgElement = document.createElement('img');
                        originalImgElement.src = e.target.result;
                        originalImgElement.style.maxWidth = '50%';
                        originalImgElement.style.maxHeight = '200px';
                        originalImgElement.style.width = img.width > img.height ? 'auto' : '100%';
                        originalImgElement.style.height = img.height > img.width ? 'auto' : '100%';
                        originalPreview.appendChild(originalImgElement);
                    };
                    img.src = e.target.result;
                };
                reader.readAsDataURL(file);
            });
        }

        function processImages() {
            const files = document.getElementById('imageInput').files;
            if (files.length === 0) {
                alert('Пожалуйста, загрузите фотографии.');
                return;
            }

            const originalPreview = document.getElementById('originalPreview');
            const processedPreview = document.getElementById('processedPreview');
            processedPreview.innerHTML = '';
            processedImages = [];
            const canvas = document.getElementById('canvas');
            const ctx = canvas.getContext('2d');

            Array.from(files).forEach(file => {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = new Image();
                    img.onload = function() {
                        canvas.width = 720;
                        canvas.height = 480;
                        ctx.filter = 'blur(10px)';
                        ctx.drawImage(img, 0, 0, 720, 480);

                        const imgWidth = img.width;
                        const imgHeight = img.height;
                        const scale = Math.min(720 / imgWidth, 480 / imgHeight);
                        const x = (720 - imgWidth * scale) / 2;
                        const y = (480 - imgHeight * scale) / 2;

                        ctx.filter = 'none';
                        ctx.drawImage(img, x, y, imgWidth * scale, imgHeight * scale);

                        const processedImgElement = document.createElement('img');
                        processedImgElement.src = canvas.toDataURL('image/jpeg');
                        processedPreview.appendChild(processedImgElement);
                        processedImages.push(canvas.toDataURL('image/jpeg'));

                        if (processedImages.length > 0) {
                            document.getElementById('downloadBtn').style.display = 'block';
                        }
                    };
                    img.src = e.target.result;
                };
                reader.readAsDataURL(file);
            });
        }

        function downloadImages() {
            processedImages.forEach((imageData, index) => {
                const link = document.createElement('a');
                link.href = imageData;
                link.download = `processed_image_${index + 1}.jpg`;
                link.click();
            });
        }

        // Генерация текста
        function generateText() {
            let condition1 = document.getElementById("condition1").value.trim();
            let condition2 = document.getElementById("condition2").value.trim();
            let condition3 = document.getElementById("condition3").value.trim();
            let condition4 = document.getElementById("condition4").value.trim();
            let condition5 = document.getElementById("condition5").value.trim();

            let generatedText = `На основе введённых условий:\n\n`;

            if (condition1) generatedText += `Условие 1: ${condition1}\n`;
            if (condition2) generatedText += `Условие 2: ${condition2}\n`;
            if (condition3) generatedText += `Условие 3: ${condition3}\n`;
            if (condition4) generatedText += `Условие 4: ${condition4}\n`;
            if (condition5) generatedText += `Условие 5: ${condition5}\n`;

            generatedText += `\nГенерированный текст на основе всех условий: \n\n`;

            // Здесь можно добавить логику для генерации текста (например, через API или искусственный интеллект)
            generatedText += `Текст сгенерирован на основе условий. (Добавьте реальную логику генерации)`;

            document.getElementById("generatedText").value = generatedText;
        }
    </script>
</body>
</html>
