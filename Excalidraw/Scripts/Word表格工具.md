---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// --- 核心排版算法（自动对齐并撑开行高） ---
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

// --- 核心生成算法（纯净版） ---
async function startProcess(rows, cols) {
    const cellWidth = 150;
    const cellHeight = 45;
    ea.style.roughness = 0;           
    ea.style.strokeWidth = 1;         
    ea.style.strokeColor = "#000000"; 
    ea.style.roundness = null;        
    ea.style.fontFamily = 2;
    ea.style.textAlign = "center";
    ea.style.verticalAlign = "middle";

    let tableIds = [];
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let x = c * cellWidth;
            let y = r * cellHeight;

            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";
            
            // 直接画一个干净的矩形，双击它