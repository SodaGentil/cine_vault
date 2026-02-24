---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---

// 1. 核心整理算法
async function autoLayoutTable(tableElements) {
    if (!tableElements || tableElements.length === 0) return;
    ea.copyViewElementsToEAforEditing(tableElements);
    const elements = ea.getElements();
    const rects = elements.filter(e => e.type === "rectangle");
    if (rects.length === 0) return;

    const TOLERANCE = 10;
    rects.sort((a, b) => {
        if (Math.abs(a.y - b.y) > TOLERANCE) return a.y - b.y;
        return a.x - b.x;
    });

    let rows = [];
    let currentRow = [rects[0]];
    for (let i = 1; i < rects.length; i++) {
        if (Math.abs(rects[i].y - currentRow[0].y) <= TOLERANCE) {
            currentRow.push(rects[i]);
        } else {
            rows.push(currentRow);
            currentRow = [rects[i]];
        }
    }
    rows.push(currentRow);

    let currentY = rows[0][0].y;
    for (let row of rows) {
        let maxHeight = Math.max(...row.map(r => r.height));
        for (let rect of row) {
            rect.y = currentY;
            rect.height = maxHeight;
        }
        currentY += maxHeight;
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
            
            // 纯净生成矩形，彻底告别条形码
            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableIds.push(rectId);
        }
    }
    ea.addToGroup(tableIds);
    await ea.addElementsToView(true, true, true);
    
    // 生成后自动稍微整理一下
    setTimeout(() => {
        const elements = ea.getViewElements().filter(el => tableIds.includes(el.id));
        autoLayoutTable(elements);
    }, 500);
}

// 3. 交互与入口 (自动判断是生成新表还是整理旧表)
const selectedElements = ea.getViewSelectedElements();