---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// 1. 清除旧弹窗
const oldOverlay = document.getElementById("my-custom-table-grid");
if (oldOverlay) oldOverlay.remove();

// 2. 创建遮罩和弹窗主体
const overlay = document.createElement("div");
overlay.id = "my-custom-table-grid";
Object.assign(overlay.style, {
    position: "fixed", top: "0", left: "0", width: "100vw", height: "100vh",
    backgroundColor: "rgba(0, 0, 0, 0.4)", zIndex: "999999",
    display: "flex", alignItems: "center", justifyContent: "center"
});

const modal = document.createElement("div");
Object.assign(modal.style, {
    backgroundColor: "var(--background-primary, #ffffff)", padding: "20px",
    borderRadius: "10px", boxShadow: "0 4px 20px rgba(0,0,0,0.3)",
    display: "flex", flexDirection: "column", alignItems: "center"
});

const title = document.createElement("h3");
title.innerText = "插入 Word 风格表格";
title.style.margin = "0 0 10px 0";

const label = document.createElement("div");
label.innerText = "请在下方滑动选择";
label.style.marginBottom = "15px";
label.style.fontWeight = "bold";
label.style.color = "#ea580c";

const gridEl = document.createElement("div");
gridEl.style.display = "grid";
gridEl.style.gridTemplateColumns = "repeat(10, 26px)";
gridEl.style.gap = "4px";

let cells = [];

// --- 3. 核心生成函数：使用最基础的 addRect 和 addText ---
async function generateTable(rows, cols) {
    if (rows === 0 || cols === 0) return;
    const cellWidth = 150;
    const cellHeight = 45;
    
    // 设置基础样式
    ea.style.roughness = 0;           
    ea.style.strokeWidth = 1;         
    ea.style.strokeColor = "#000000"; 
    ea.style.roundness = null;        
    ea.style.fontFamily = 2; // 标准字体
    ea.style.textAlign = "center";
    ea.style.verticalAlign = "middle";

    let tableElements = [];

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let x = c * cellWidth;
            let y = r * cellHeight;

            // 背景色处理
            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";

            // 生成矩形格子
            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableElements.push(rectId);

            // 在格子中心生成一个隐藏的文本占位符 (防止文字溢出)
            // 这种方式兼容性最高，双击格子即可在中心输入
            let textId = ea.addText(x + cellWidth/2, y + cellHeight/2, " ", {
                width: cellWidth - 10,
                textAlign: "center",
                verticalAlign: "middle",
                box: true // 限制在框内
            });
            tableElements.push(textId);
        }
    }

    ea.addToGroup(tableElements);
    await ea.addElementsToView(true, true, true);
}

// 4. 构建 10x10 网格
for (let r = 1; r <= 10; r++) {
    for (let c = 1; c <= 10; c++) {
        const cell = document.createElement("div");
        Object.assign(cell.style, {
            width: "26px", height: "26px", cursor: "pointer", transition: "all 0.1s",
            border: "1px solid var(--background-modifier-border, #ccc)", borderRadius: "2px"
        });
        
        cell.dataset.row = r;
        cell.dataset.col = c;

        cell.addEventListener("mouseenter", () => {
            label.innerText = `${c} 列 × ${r} 行 表格`;
            cells.forEach(item => {
                const ir = parseInt(item.dataset.row);
                const ic = parseInt(item.dataset.col);
                const isSelected = ir <= r && ic <= c;
                item.style.backgroundColor = isSelected ? "#fed7aa" : "transparent";
                item.style.borderColor = isSelected ? "#ea580c" : "var(--background-modifier-border, #ccc)";
            });
        });

        cell.addEventListener("click", () => {
            overlay.remove();
            generateTable(r, c);
        });

        cells.push(cell);
        gridEl.appendChild(cell);
    }
}

overlay.addEventListener("click", (e) => { if (e.target === overlay) overlay.remove(); });
modal.append(title, label, gridEl);
overlay.appendChild(modal);
document.body.appendChild(overlay);