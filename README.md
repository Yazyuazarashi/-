# -<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>戦力合計ツール（複数画像対応）</title>
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4"></script>
  <style>
    body { font-family: sans-serif; padding: 20px; }
    #dropzone {
      border: 2px dashed #aaa;
      padding: 30px;
      text-align: center;
      margin-bottom: 20px;
      background: #f9f9f9;
    }
  </style>
</head>
<body>

  <h2>戦力集計ツール（複数画像対応）</h2>
  <p>画像を選択するか、下の枠にドラッグ＆ドロップしてね！</p>

  <input type="file" id="imageInput" accept="image/*" multiple />
  <div id="dropzone">ここに画像をドロップ！</div>

  <pre id="output">読み込み中のログがここに表示されます…</pre>

  <script>
    const imageInput = document.getElementById('imageInput');
    const dropzone = document.getElementById('dropzone');
    const output = document.getElementById('output');

    dropzone.addEventListener('dragover', (e) => {
      e.preventDefault();
      dropzone.style.background = '#e0f7ff';
    });

    dropzone.addEventListener('dragleave', () => {
      dropzone.style.background = '#f9f9f9';
    });

    dropzone.addEventListener('drop', (e) => {
      e.preventDefault();
      dropzone.style.background = '#f9f9f9';
      const files = Array.from(e.dataTransfer.files);
      processImages(files);
    });

    imageInput.addEventListener('change', () => {
      const files = Array.from(imageInput.files);
      processImages(files);
    });

    function processImages(files) {
      let totalSum = 0;
      let allNumbers = [];
      let processed = 0;

      output.innerText = `読み込み中...（${files.length}枚）\n`;

      files.forEach(file => {
        Tesseract.recognize(file, 'eng+jpn', { logger: m => console.log(m) })
          .then(({ data: { text } }) => {
            const lines = text.split('\n');

            const numbers = lines
              .filter(line => line.includes('戦力'))
              .map(line => {
                const match = line.match(/\d[\d,]*/);
                return match ? parseInt(match[0].replace(/,/g, '')) : 0;
              });

            totalSum += numbers.reduce((a, b) => a + b, 0);
            allNumbers.push(...numbers);
          })
          .finally(() => {
            processed++;
            if (processed === files.length) {
              output.innerText = 
                `抽出された戦力:\n${allNumbers.join(', ')}\n\n合計戦力: ${totalSum.toLocaleString()}`;
            }
          });
      });
    }
  </script>
</body>
</html>
読み込みプログラム
