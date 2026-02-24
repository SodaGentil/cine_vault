---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// === Excalidraw Word表格生成与【网格轨道】排版引擎 ===

// 1. 核心排版引擎：基于矩阵轨道的二维自适应
async function autoLayoutTable(tableElements) {
    if (!tableElements || tableElements.length === 0) return;
    ea.copyViewElementsToEAforEditing(tableElements);
    const rects = ea.getElements().filter(e => e.type === "rectangle");
    if (rects.length === 0) return;

    // 容差值：允许手动拖拽时产生 20px 以内的对齐误差
    const TOLERANCE = 20; 

    // 步骤 A：提取唯一的 X 和 Y 坐标，形成“网格轨道参考线”
    let xs = [...new Set(rects.map(r => r.x))].sort((a, b) => a - b);
    let ys = [...new Set(rects.map(r => r.y))].sort((a, b) => a - b);

    let colTracks = [], rowTracks = [];
    xs.forEach(x => { if (colTracks.length === 0 || x - colTracks[colTracks.length - 1] > TOLERANCE) colTracks.push(x); });
    ys.forEach(y => { if (rowTracks.length === 0 || y - rowTracks[rowTracks.length - 1] > TOLERANCE) rowTracks.push(y); });

    // 步骤 B：初始化网格矩阵，并记录每列最大宽度、每行最大高度
    let colWidths = new Array(colTracks.length).fill(0);
    let rowHeights = new Array(rowTracks.length).fill(0);
    let grid = Array(rowTracks.length).fill(null).map(() => Array(colTracks.length).fill(null));

    // 步骤 C：将所有格子对号入座，放入矩阵
    for (let rect of rects) {
        let c = colTracks.findIndex(cx => Math.abs(rect.x - cx) <= TOLERANCE);
        let r = rowTracks.findIndex(ry => Math.abs(rect.y - ry) <= TOLERANCE);
        if (c !== -1 && r !== -1) {
            grid[r][c] = rect;
            colWidths[c] = Math.max(colWidths[c], rect.width);
            rowHeights[r] = Math.max(rowHeights[r], rect.height);
        }
    }

    // 步骤 D：基于矩阵轨道，重新计算绝对坐标并渲染
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
            currentX += colWidths[c]; // X 轴向右推进
        }
        currentY += rowHeights[r]; // Y 轴向下推进
    }

    await ea.addElementsToView(false, false);
}

// 2. 生成网格算法
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
            let x = c * cellWidth;
            let y = r * cellHeight;
            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";
            
            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableIds.push(rectId);
        }
    }
    ea.addToGroup(tableIds);
    await ea.addElementsToView(true, true, true);
    
    // 生成后自动整理坐标
    setTimeout(() => {
        const elements = ea.getViewElements().filter(el => tableIds.includes(el.id));
        autoLayoutTable(elements);
    }, 500);
}

// 3. 交互与入口
const selectedElements = ea.getViewSelectedElements();

if (selectedElements.length > 0) {
    autoLayoutTable(selectedElements).then(() => {
        new ea.obsidian.Notice("✨ 网格轨道已重组！表格长宽自适应对齐。");
    });
} else {
    const oldOverlay = document.getElementById("integrated-table-grid");
    if (oldOverlay) oldOverlay.remove();

    const overlay = document.createElement("div");
    overlay.id = "integrated-table-grid";
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

    const label = document.createElement("div");
    label.innerText = "请滑动选择表格大小";
    label.style.marginBottom = "15px";
    label.style.fontWeight = "bold";
    label.style.color = "#ea580c";

    const gridEl = document.createElement("div");
    gridEl.style.display = "grid";
    gridEl.style.gridTemplateColumns = "repeat(10, 26px)";
    gridEl.style.gap = "4px";

    let cells = [];
    for (let r = 1; r <= 10; r++) {
        for (let c = 1; c <= 10; c++) {
            const cell = document.createElement("div");
            Object.assign(cell.style, {
                width: "26px", height: "26px", cursor: "pointer",
                border: "1px solid var(--background-modifier-border, #ccc)", borderRadius: "2px"
            });
            cell.dataset.row = r; cell.dataset.col = c;
            cell.addEventListener("mouseenter", () => {
                label.innerText = `${c} 列 × ${r} 行 表格`;
                cells.forEach(item => {
                    const isSel = parseInt(item.dataset.row) <= r && parseInt(item.dataset.col) <= c;
                    item.style.backgroundColor = isSel ? "#fed7aa" : "transparent";
                    item.style.borderColor = isSel ? "#ea580c" : "var(--background-modifier-border, #ccc)";
                });
            });
            cell.addEventListener("click", () => { overlay.remove(); startProcess(r, c); });
            cells.push(cell);
            gridEl.appendChild(cell);
        }
    }
    overlay.addEventListener("click", (e) => { if (e.target === overlay) overlay.remove(); });
    modal.appendChild(label);
    modal.appendChild(gridEl);
    overlay.appendChild(modal);
    document.body.appendChild(overlay);
}