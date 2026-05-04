<！DOCTYPE html>
<超文本标记语言朗="zh-CN">
<头>
原字符集="utf-8">
<元姓名="视口" 内容="宽度=设备宽度，初始比例=1.0">
<标题>万能资源站｜拖拽上传+网页视频解析播放</标题>
<风格>
*{边缘：0；填充：0；箱体尺寸：边框框；}
身体{字体系列：system-ui，微软雅黑，无芯线；background：#f5f6f8；padding:20px；}
.容器{最大宽度：1200px；边缘：0自动；}

/* 网址视频区域 */
.url-box{
background：#fff；
填料：20px；
边界半径：12px；
底边距：20px；
box-shadow:02px8px rgba(0,0,0,0.08);
}
.url-box h3{margin-bottom:12px;color:#333;}
.url-input{
  width:100%;
  padding:12px 15px;
  border:1px solid #ddd;
  border-radius:8px;
  font-size:14px;
  margin-bottom:10px;
}
.url-btn{
  background:#2478f5;
  color:#fff;
  border:none;
  padding:10px 20px;
  border-radius:6px;
  cursor:pointer;
  margin-right:10px;
}
.clear-url-btn{
  background:#f56c6c;
  color:#fff;
  border:none;
  padding:10px 20px;
  border-radius:6px;
  cursor:pointer;
}
.tip{color:#666;font-size:13px;margin-top:6px;}

/* 拖拽上传区 */
#dropBox{
  border:3px dashed #409eff;
  border-radius:12px;
  text-align:center;
  padding:50px 20px;
  margin-bottom:20px;
  transition:0.3s;
  background:#fff;
}
#dropBox.active{
  background:#ecf5ff;
  border-color:#2478f5;
}
#dropBox p{color:#666;margin:8px 0;}
#fileBtn{
  background:#409eff;
  color:#fff;
  border:none;
  padding:10px 24px;
  border-radius:6px;
  cursor:pointer;
  font-size:15px;
}
input[type="file"]{display:none;}

/* 资源列表 */
.item{
  background:#fff;
  padding:15px;
  border-radius:8px;
  margin-bottom:12px;
  display:flex;
  align-items:center;
  justify-content:space-between;
  box-shadow:0 2px 6px #00000010;
}
.name{flex:1;margin:0 15px;word-break:break-all;}
.btn-group button{
  margin:0 4px;
  padding:6px 12px;
  border:none;
  border-radius:4px;
  cursor:pointer;
}
.preview-btn{background:#67c23a;color:#fff;}
.down-btn{background:#e6a23c;color:#fff;}
.del-btn{background:#f56c6c;color:#fff;}

/* 播放器弹窗 */
.modal{
  position:fixed;
  top:0;left:0;
  width:100%;height:100%;
  background:#000000dd;
  display:none;
  align-items:center;
  justify-content:center;
  z-index:999;
  padding:20px;
}
.modal-content{
  width:100%;
  max-width:1000px;
}
.close{
  position:absolute;
  top:30px;right:30px;
  color:#fff;
  font-size:36px;
  cursor:pointer;
  user-select:none;
}
iframe{
  width:100%;
  height:600px;
  border-radius:8px;
  border:none;
}
video{
  width:100%;
  border-radius:8px;
  outline:none;
}
</style>
</head>
<body>
<div class="container">
  <!-- 网页视频播放 + 一键清空网址 -->
  <div class="url-box">
    <h3>🌐 全网视频链接播放</h3>
    <input class="url-input" id="videoUrl" placeholder="粘贴任意视频网页链接 / 视频直链">
    <button class="url-btn" id="loadUrlVideo">打开播放</button>
    <button class="clear-url-btn" id="clearUrl">一键清空网址</button>
    <div class="tip">普通视频网页、MP4直链均可播放，支持拖拽进度</div>
  </div>

  <!-- 本地文件拖拽 -->
  <div id="dropBox">
    <h2>📁 拖拽本地文件到此处</h2>
    <p>视频 / 图片 / 文档 预览+下载</p>
    <button id="fileBtn">手动选择文件</button>
    <input type="file" id="fileInput" multiple>
  </div>

  <!-- 资源列表 -->
  <div id="list"></div>
</div>

<!-- 播放弹窗 -->
<div class="modal" id="modal">
  <span class="close" id="closeModal">×</span>
  <div class="modal-content" id="modalBody"></div>
</div>

<script>
const dropBox = document.getElementById('dropBox');
const fileInput = document.getElementById('fileInput');
const fileBtn = document.getElementById('fileBtn');
const list = document.getElementById('list');
const modal = document.getElementById('modal');
const modalBody = document.getElementById('modalBody');
const closeModal = document.getElementById('closeModal');
const videoUrl = document.getElementById('videoUrl');
const loadUrlVideo = document.getElementById('loadUrlVideo');
const clearUrl = document.getElementById('clearUrl');

let fileList = [];

// 一键清空输入框网址
clearUrl.onclick = function(){
  videoUrl.value = "";
  videoUrl.focus();
}

// 网页链接播放
loadUrlVideo.onclick = function(){
  let url = videoUrl.value.trim();
  if(!url){
    alert('请先粘贴视频链接');
    return;
  }
  modalBody.innerHTML = `<iframe src="${url}" allowfullscreen></iframe>`;
  modal.style.display = 'flex';
}
videoUrl.addEventListener('keydown',e=>{
  if(e.key === 'Enter') loadUrlVideo.click();
})

// 本地文件选择
fileBtn.onclick = () => fileInput.click();
fileInput.onchange = (e) => {
  const files = e.target.files;
  if(files.length) handleFiles(files);
  fileInput.value = '';
};

// 拖拽事件
dropBox.addEventListener('dragover', e=>{e.preventDefault();dropBox.classList.add('active');});
dropBox.addEventListener('dragleave', ()=>dropBox.classList.remove('active'));
dropBox.addEventListener('drop', e=>{
  e.preventDefault();
  dropBox.classList.remove('active');
  let files = e.dataTransfer.files;
  if(files.length) handleFiles(files);
});

// 处理本地文件
function handleFiles(files){
  for(let file of files){
    let url = URL.createObjectURL(file);
    fileList.push({
      name: file.name,
      type: file.type,
      url: url,
      raw: file
    });
  }
  render();
}

// 渲染列表
function render(){
  if(fileList.length === 0){
    list.innerHTML = '<div style="text-align:center;color:#999;padding:40px;">暂无资源</div>';
    return;
  }
  let html = '';
  fileList.forEach((item,idx)=>{
    html += `
    <div class="item">
      <div class="name">${item.name}</div>
      <div class="btn-group">
        <button class="preview-btn" onclick="openPre(${idx})">在线预览</button>
        <a href="${item.url}" download="${item.name}"><button class="down-btn">下载</button></a>
        <button class="delBtn" onclick="delItem(${idx})">删除</button>
      </div>
    </div>
    `;
  });
  list.innerHTML = html;
}

// 本地文件预览
window.openPre = function(idx){
  const item = fileList[idx];
  let html = '';
  if(item.type.includes('video')){
    html = `<video controls src="${item.url}"></video>`;
  }else if(item.type.includes('image')){
    html = `<img src="${item.url}" style="max-width:100%;border-radius:8px;">`;
  }else{
    html = '<div style="padding:30px;text-align:center;color:#666;">该文件请下载查看</div>';
  }
  modalBody.innerHTML = html;
  modal.style.display = 'flex';
}

// 关闭弹窗
closeModal.onclick = ()=> modal.style.display = 'none';
modal.onclick = e=>{if(e.target === modal) modal.style.display = 'none';};

// 删除文件
window.delItem = function(idx){
  URL.revokeObjectURL(fileList[idx].url);
  fileList.splice(idx,1);
  render();
}
</script>
</body>
</html>
