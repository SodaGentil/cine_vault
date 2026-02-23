const input = await utils.inputPrompt(
  "📝 插入 Word 风格表格",
  "请输入【行数,列数】（例如 4,3 代表4行3列）：",
  "4,3"
);

if (!input) return;

const parts = input.split(/[,，xX\s*]+/);
const rows = parseInt(parts[0]);
const cols = parseInt(parts[1]);

if (isNaN(rows) || isNaN(cols) || rows <= 0 || cols <= 0) {
    new Notice("输入有误！请输入纯数字和分隔符，如 4,3");
    return;
}

// 设定类似 Word 的默认长宽比
const cellWidth = 150;
const cellHeight = 40;

// 核心优化：Word 风格样式设置
ea.style.roughness = 0;           // 笔直的线条
ea.style.strokeWidth = 1;         // 细边框
ea.style.strokeColor = "#000000"; // 纯黑边框
ea.style.roundness = null;        // 直角
ea.style.fontFamily = 2;          // 强制设置为标准字体 (1:手写, 2:标准, 3:等宽)

let tableElements = [];

for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
        let x = c * cellWidth;
        let y = r * cellHeight;

        if (r === 0) {
            // 表头：浅灰色实体填充
            ea.style.backgroundColor = "#f3f4f6"; 
            ea.style.fillStyle = "solid";
        } else {
            // 内容：纯白色实体填充（防止背景穿模）
            ea.style.backgroundColor = "#ffffff"; 
            ea.style.fillStyle = "solid"; 
        }

        let rectId = ea.addRect(x, y, cellWidth, cellHeight);
        tableElements.push(rectId);
    }
}

// 自动打组，防止散架
ea.addToGroup(tableElements);

await ea.addElementsToView(true, true, true);
new Notice(`✅ Word 风格 ${rows}×${cols} 表格插入成功！`);