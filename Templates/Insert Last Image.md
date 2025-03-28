<%*
const fs = require('fs');
const path = require('path');

// images 폴더 절대 경로 구성
const vaultPath = app.vault.adapter.basePath;
const imagesPath = path.join(vaultPath, '../images');

const files = fs.readdirSync(imagesPath)
  .filter(file => file.endsWith('.png'))
  .map(file => ({
    name: file,
    time: fs.statSync(path.join(imagesPath, file)).mtime.getTime()
  }))
  .sort((a, b) => b.time - a.time); // 최신순 정렬

const latest = files[0]?.name;

if (latest) {
  tR += `![](../images/${latest})`;
} else {
  tR += `⚠️ No image found in images folder`;
}
%>
