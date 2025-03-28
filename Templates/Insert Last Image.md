<%*
const fs = require('fs');
const path = require('path');

const vaultPath = app.vault.adapter.basePath;
const imagesFolder = path.join(vaultPath, 'images');

try {
  const imageFiles = fs.readdirSync(imagesFolder)
    .filter(file => file.endsWith('.png'))
    .map(file => ({
      name: file,
      time: fs.statSync(path.join(imagesFolder, file)).mtime.getTime()
    }))
    .sort((a, b) => b.time - a.time);

  const latest = imageFiles[0]?.name;

  if (latest) {
    tR += `![](../images/${encodeURIComponent(latest)})`;
  } else {
    tR += `⚠️ images 폴더에 .png 파일이 없습니다`;
  }
} catch (err) {
  tR += `🚫 오류 발생: ${err.message}`;
}
%>
