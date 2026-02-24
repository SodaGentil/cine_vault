---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// === Excalidraw 表格引擎 + 悬浮外挂按钮 ===

// 1. 核心排版引擎：基于矩阵轨道的二维自适应
async function autoLayoutTable(tableElements) {
    if (!tableElements || tableElements.length === 0) return;
    ea.copyViewElementsToEAforEditing(tableElements);
    const rects = ea.getElements().filter(e => e.type === "rectangle");
    if (rects.length === 0) return;

    const TOLERANCE = 20; 
    let xs = [...new Set(rects.map(r => r.x))].sort((a, b) => a - b);
    let ys = [...new Set(rects.map(r => r.y))].sort((a, b) => a - b);

    let colTracks = [], rowTracks = [];
    xs.forEach(x => { if (colTracks.length === 0 || x - colTracks[colTracks.length - 1] > TOLERANCE) colTracks.push(x); });
    ys.forEach(y => { if (rowTracks.length === 0 || y - rowTracks[rowTracks.length - 1] > TOLERANCE) rowTracks.push(y); });

    let colWidths = new Array(colTracks.length).fill(0);
    let rowHeights = new Array(rowTracks.length).fill(0);
    let grid = Array(rowTracks.length).fill(null).map(() => Array(colTracks.length).fill(null));

    for (let rect of rects) {
        let c = colTracks.findIndex(cx => Math.abs(rect.x - cx) <= TOLERANCE);
        let r = rowTracks.findIndex(ry => Math.abs(rect.y - ry) <= TOLERANCE);
        if (c !== -1 && r !== -1) {
            grid[r][c] = rect;
            colWidths[c] = Math.max(colWidths[c], rect.width);
            rowHeights[r] = Math.max(rowHeights[r], rect.height);
        }
    }

    let startY = Math.min(...rects.map(r => r.y));
    let startX = Math.min(...rects.map(r => r.x));
    let currentY = startY;

    for (let r = 0; r < rowTracks.length; r++) {
        let currentX = startX;
        for (let c = 0; c < colTracks.length; c++) {
            let cell = grid[r][c];
            if (cell) {
                cell.x = currentX;
                cell.y = currentY;
                cell.width = colWidths[c];
                cell.height = rowHeights[r];
            }
            currentX += colWidths[c]; 
        }
        currentY += rowHeights[r]; 
    }

    await ea.addElementsToView(false, false);
}

// 2. 注入全局悬浮外挂按钮 (Floating Action Button)
function injectFloatingButton() {
    if (document.getElementById("excalidraw-table-reflow-btn")) return;

    const btn = document.createElement("button");
    btn.id = "excalidraw-table-reflow-btn";
    btn.innerText = "🪄 对齐表格";
    
    // 按钮的 UI 样式，固定在画布右下角
    Object.assign(btn.style, {
        position: "absolute", bottom: "30px", right: "30px",
        padding: "12px 20px", backgroundColor: "#ea580c", color: "white",
        border: "none", borderRadius: "8px", fontWeight: "bold",
        cursor: "pointer", zIndex: "99999", boxShadow: "0 4px 12px rgba(0,0,0,0.3)",
        transition: "transform 0.1s"
    });

    btn.onmousedown = () => btn.style.transform = "scale(0.95)";
    btn.onmouseup = () => btn.style.transform = "scale(1)";

    // 点击按钮直接执行排版
    btn.onclick = () => {
        const selected = ea.getViewSelectedElements();
        if (selected.length > 0) {
            autoLayoutTable(selected).then(() => {
                new ea.obsidian.Notice("✨ 表格长宽已自动对齐！");
            });
        } else {
            new ea.obsidian.Notice("⚠️ 请先单击选中你要对齐的表格！");
        }
    };

    document.body.appendChild(btn);
}

// 3. 生成表格的交互网格
async function startProcess(rows, cols) {
    const cellWidth = 150;
    const cellHeight = 45;
    ea.style.roughness = 0;
    ea.style.strokeWidth = 1;
    ea.style.strokeColor = "#000000";
    ea.style.roundness = null;
    ea.style.fontFamily = 2;

    let tableIds = [];
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";
            let rectId = ea.addRect(c * cellWidth, r * cellHeight, cellWidth, cellHeight);
            tableIds.push(rectId);
        }
    }