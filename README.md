<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>圖片裁切＋去背工具</title>
  <style>
    body { font-family: sans-serif; text-align: center; padding: 2rem; }
    canvas, img { max-width: 300px; margin-top: 1rem; }
    input[type="file"] { margin-bottom: 1rem; }
  </style>
</head>
<body>
  <h1>圖片裁切＋去背工具</h1>
  <input type="file" id="upload" accept="image/*" />
  <br/>
  <canvas id="canvas"></canvas>
  <br/>
  <button id="removeBgBtn">去背</button>
  <br/>
  <img id="result" alt="處理後圖片" />

  <script>
    const upload = document.getElementById('upload');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const resultImg = document.getElementById('result');
    const removeBgBtn = document.getElementById('removeBgBtn');

    let imageDataURL = '';

    upload.addEventListener('change', (e) => {
      const file = e.target.files[0];
      const reader = new FileReader();

      reader.onload = function (event) {
        const img = new Image();
        img.onload = function () {
          const size = Math.min(img.width, img.height);
          canvas.width = size;
          canvas.height = size;
          ctx.drawImage(img,
            (img.width - size) / 2,
            (img.height - size) / 2,
            size, size,
            0, 0,
            size, size);

          imageDataURL = canvas.toDataURL('image/png');
        };
        img.src = event.target.result;
      };

      if (file) reader.readAsDataURL(file);
    });

    removeBgBtn.addEventListener('click', async () => {
      if (!imageDataURL) return alert("請先上傳圖片！");

      const base64 = imageDataURL.split(",")[1];

      const response = await fetch("https://api.remove.bg/v1.0/removebg", {
        method: "POST",
        headers: {
          "X-Api-Key": "YOUR_API_KEY_HERE",
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ image_file_b64: base64, size: "auto" })
      });

      if (!response.ok) {
        const error = await response.text();
        alert("去背失敗：" + error);
        return;
      }

      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      resultImg.src = url;
    });
  </script>
</body>
</html>
