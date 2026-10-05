# Create Mandala Tool Easy: Draw Online!
MandalaTool is a free, web-based mandala creation suite designed for sketching, doodling, and collaborating on intricate digital art. Developed as a tool for creating "Zen Geometry," the platform requires no software downloads and functions entirely within your web browser.
**Core Features of the Platform**
-

**View Demo**: [Mandala Generator Tool](https://coloringfiles.com/creating-mandala.html)

-
<img width="1321" height="947" alt="Image" src="https://github.com/user-attachments/assets/877dee3a-9508-4446-8182-4cb08314d314" />

The suite offers an extensive array of drawing guides that go beyond traditional circular art. Users can instantly switch between several complex geometry layouts:
• **Mandala Radial Symmetry**: Automatically mirrors strokes across multiple custom axes to build highly balanced, kaleidoscopic patterns.
• **Tessellation & Repetitions**: Allows artists to repeat complex interlaced patterns seamlessly across the canvas.
• **Recursion Engine**: Supports recursive drawing pathways, perfect for generating complex fractures or botanical plant styles.
• **Dedicated Design Modes**: Built-in specific guides designed directly for tailoring snowflakes or organic plant geometry.

More [Mandala coloring page](https://coloringfiles.com/mandala-coloring-pages) for example you mind.
//
Sciprt codes;
 <script>
    /*
     * Mandala Creator Studio Core Logic
     * Features:
     * - Math Radial Symmetry Engine with optional Mirror Reflection
     * - Dual Canvas Architecture (Main artwork + Overlay guides/preview)
     * - DoS Hardening: Input clamping, RAF throttling, Stack Capping (max 20 states)
     * - Clean PDF Export via jsPDF
     */

    // --- State Management ---
    const STATE = {
      theme: 'light',
      tool: 'pen', // 12 tools: pen, circle, square, rectangle, oval, star, flower, polygon, heart, diamond, spiral, eraser
      axes: 12,
      enableRadial: true,
      mirrorMode: true,
      showGuides: true,
      strokeColor: '#000000',
      bgColor: '#ffffff',
      brushSize: 4,
      opacity: 1.0,
      fillMode: 'stroke', // 'stroke' or 'fill'
      isDrawing: false,
      startPoint: { x: 0, y: 0 },
      lastPoint: { x: 0, y: 0 },
      currentPoint: { x: 0, y: 0 },
      history: [],
      historyIndex: -1,
      maxHistory: 20 // Memory exhaustion protection cap
    };

    // --- DOM Element References ---
    const mainCanvas = document.getElementById('mainCanvas');
    const mainCtx = mainCanvas.getContext('2d', { willReadFrequently: true });

    const guideCanvas = document.getElementById('guideCanvas');
    const guideCtx = guideCanvas.getContext('2d');

    const themeSelect = document.getElementById('themeSelect');
    const btnReset = document.getElementById('btnReset');
    const btnUndo = document.getElementById('btnUndo');
    const btnRedo = document.getElementById('btnRedo');
    const btnExportPdf = document.getElementById('btnExportPdf');

    const sliderAxes = document.getElementById('sliderAxes');
    const numAxes = document.getElementById('numAxes');
    const axesValue = document.getElementById('axesValue');
    const chkShowGuides = document.getElementById('chkShowGuides');
    const chkEnableRadial = document.getElementById('chkEnableRadial');
    const chkMirrorMode = document.getElementById('chkMirrorMode');

    const pickerStroke = document.getElementById('pickerStroke');
    const textStrokeHex = document.getElementById('textStrokeHex');
    const pickerBg = document.getElementById('pickerBg');
    const textBgHex = document.getElementById('textBgHex');

    const sliderBrushSize = document.getElementById('sliderBrushSize');
    const valBrushSize = document.getElementById('valBrushSize');
    const sliderOpacity = document.getElementById('sliderOpacity');
    const valOpacity = document.getElementById('valOpacity');

    const toolButtons = document.querySelectorAll('.tool-card');
    const fillModeRadios = document.querySelectorAll('input[name="fillMode"]');
    const coordDisplay = document.getElementById('coordDisplay');
    const historyStatus = document.getElementById('historyStatus');

    // --- Input Sanitization & Security Helpers ---
    function clamp(value, min, max) {
      return Math.min(Math.max(Number(value) || min, min), max);
    }

    function isValidHexColor(hex) {
      return /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/.test(hex);
    }

    // --- Theme Switcher Logic ---
    function setTheme(newTheme) {
      STATE.theme = newTheme;
      document.body.setAttribute('data-theme', newTheme);
      themeSelect.value = newTheme;
    }

    themeSelect.addEventListener('change', (e) => setTheme(e.target.value));

    // --- Canvas Setup & Initialization ---
    function initCanvas() {
      // Set background initial color
      fillCanvasBackground(STATE.bgColor);
      // Save pristine state into history stack
      saveCanvasState();
      // Render radial symmetry overlay guidelines
      drawGuideOverlay();
    }

    function fillCanvasBackground(colorHex) {
      mainCtx.save();
      mainCtx.fillStyle = colorHex;
      mainCtx.fillRect(0, 0, mainCanvas.width, mainCanvas.height);
      mainCtx.restore();
    }

    // --- History Stack Operations (Undo / Redo with RAM Cap) ---
    function saveCanvasState() {
      // Truncate redo states if drawing fresh action
      if (STATE.historyIndex < STATE.history.length - 1) {
        STATE.history = STATE.history.slice(0, STATE.historyIndex + 1);
      }

      // Enforce history stack size cap to avoid memory leak attack
      if (STATE.history.length >= STATE.maxHistory) {
        STATE.history.shift();
      } else {
        STATE.historyIndex++;
      }

      const imgData = mainCtx.getImageData(0, 0, mainCanvas.width, mainCanvas.height);
      STATE.history.push(imgData);
      updateHistoryUI();
    }

    function undo() {
      if (STATE.historyIndex > 0) {
        STATE.historyIndex--;
        const snapshot = STATE.history[STATE.historyIndex];
        mainCtx.putImageData(snapshot, 0, 0);
        updateHistoryUI();
        drawGuideOverlay();
      }
    }

    function redo() {
      if (STATE.historyIndex < STATE.history.length - 1) {
        STATE.historyIndex++;
        const snapshot = STATE.history[STATE.historyIndex];
        mainCtx.putImageData(snapshot, 0, 0);
        updateHistoryUI();
        drawGuideOverlay();
      }
    }

    function resetMandala() {
      if (confirm('Are you sure you want to reset the mandala canvas? All unsaved work will be cleared.')) {
        STATE.history = [];
        STATE.historyIndex = -1;
        fillCanvasBackground(STATE.bgColor);
        saveCanvasState();
        drawGuideOverlay();
      }
    }

    function updateHistoryUI() {
      btnUndo.disabled = STATE.historyIndex <= 0;
      btnRedo.disabled = STATE.historyIndex >= STATE.history.length - 1;
      historyStatus.textContent = `State: ${STATE.historyIndex + 1} / ${STATE.history.length}`;
    }

    btnUndo.addEventListener('click', undo);
    btnRedo.addEventListener('click', redo);
    btnReset.addEventListener('click', resetMandala);

    // Keyboard Shortcuts (Ctrl+Z, Ctrl+Y)
    window.addEventListener('keydown', (e) => {
      if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'z') {
        if (e.shiftKey) redo();
        else undo();
      } else if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'y') {
        redo();
      }
    });

    // --- Radial Guidelines Rendering ---
    function drawGuideOverlay() {
      guideCtx.clearRect(0, 0, guideCanvas.width, guideCanvas.height);

      if (!STATE.showGuides) return;

      const centerX = guideCanvas.width / 2; // 600
      const centerY = guideCanvas.height / 2; // 600
      const radius = Math.min(centerX, centerY);

      guideCtx.save();
      guideCtx.strokeStyle = STATE.theme === 'light' ? 'rgba(0, 0, 0, 0.15)' : 'rgba(255, 255, 255, 0.15)';
      guideCtx.lineWidth = 1;
      guideCtx.setLineDash([4, 4]);

      // Concentric guide circles
      for (let r = 100; r <= radius; r += 100) {
        guideCtx.beginPath();
        guideCtx.arc(centerX, centerY, r, 0, Math.PI * 2);
        guideCtx.stroke();
      }

      // Radial slice guide lines
      if (STATE.enableRadial) {
        const axes = STATE.axes;
        const angleStep = (Math.PI * 2) / axes;

        for (let i = 0; i < axes; i++) {
          const angle = i * angleStep;
          const x2 = centerX + Math.cos(angle) * radius;
          const y2 = centerY + Math.sin(angle) * radius;

          guideCtx.beginPath();
          guideCtx.moveTo(centerX, centerY);
          guideCtx.lineTo(x2, y2);
          guideCtx.stroke();
        }
      }

      guideCtx.restore();
    }

    // --- Mathematical Radial Symmetry Canvas Transformer ---
    /*
     * Replicates any drawing path or shape function across all N radial symmetry sectors,
     * with optional mirror reflection.
     */
    function executeRadialSymmetry(ctx, drawFn) {
      const centerX = 600;
      const centerY = 600;
      const axes = STATE.enableRadial ? STATE.axes : 1;
      const angleStep = (Math.PI * 2) / axes;

      for (let i = 0; i < axes; i++) {
        // Primary Radial Sector
        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(i * angleStep);
        drawFn(ctx);
        ctx.restore();

        // Mirrored Reflection Sector
        if (STATE.enableRadial && STATE.mirrorMode) {
          ctx.save();
          ctx.translate(centerX, centerY);
          ctx.rotate(i * angleStep);
          ctx.scale(1, -1);
          drawFn(ctx);
          ctx.restore();
        }
      }
    }

    // --- 12 Geometry Drawing Tool Routines ---
    // All coordinates inside drawFn are relative to canvas center (0, 0)
    function renderToolShape(ctx, tool, p1, p2, isFill) {
      const dx = p2.x - p1.x;
      const dy = p2.y - p1.y;
      const dist = Math.hypot(dx, dy);

      ctx.beginPath();

      switch (tool) {
        case 'circle': {
          ctx.arc(p1.x, p1.y, dist, 0, Math.PI * 2);
          break;
        }

        case 'square': {
          const side = Math.max(Math.abs(dx), Math.abs(dy)) * 2;
          ctx.rect(p1.x - side / 2, p1.y - side / 2, side, side);
          break;
        }

        case 'rectangle': {
          ctx.rect(p1.x, p1.y, dx, dy);
          break;
        }

        case 'oval': {
          ctx.ellipse(p1.x, p1.y, Math.abs(dx) || 1, Math.abs(dy) || 1, 0, 0, Math.PI * 2);
          break;
        }

        case 'star': {
          const points = 5;
          const outerR = dist;
          const innerR = dist * 0.4;
          for (let i = 0; i < points * 2; i++) {
            const r = i % 2 === 0 ? outerR : innerR;
            const a = (i * Math.PI) / points - Math.PI / 2;
            const x = p1.x + Math.cos(a) * r;
            const y = p1.y + Math.sin(a) * r;
            if (i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
          }
          ctx.closePath();
          break;
        }

        case 'flower': {
          const petals = 6;
          for (let i = 0; i < petals; i++) {
            const angle = (i * 2 * Math.PI) / petals;
            const px = p1.x + Math.cos(angle) * dist;
            const py = p1.y + Math.sin(angle) * dist;
            ctx.moveTo(p1.x, p1.y);
            ctx.quadraticCurveTo(
              p1.x + Math.cos(angle + 0.5) * (dist * 1.2),
              p1.y + Math.sin(angle + 0.5) * (dist * 1.2),
              px,
              py
            );
            ctx.quadraticCurveTo(
              p1.x + Math.cos(angle - 0.5) * (dist * 1.2),
              p1.y + Math.sin(angle - 0.5) * (dist * 1.2),
              p1.x,
              p1.y
            );
          }
          break;
        }

        case 'polygon': { // Hexagon
          const sides = 6;
          for (let i = 0; i < sides; i++) {
            const a = (i * 2 * Math.PI) / sides;
            const x = p1.x + Math.cos(a) * dist;
            const y = p1.y + Math.sin(a) * dist;
            if (i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
          }
          ctx.closePath();
          break;
        }

        case 'heart': {
          const scale = Math.max(dist / 30, 0.5);
          ctx.save();
          ctx.translate(p1.x, p1.y);
          ctx.beginPath();
          ctx.moveTo(0, 0);
          ctx.bezierCurveTo(-15 * scale, -20 * scale, -30 * scale, 10 * scale, 0, 30 * scale);
          ctx.bezierCurveTo(30 * scale, 10 * scale, 15 * scale, -20 * scale, 0, 0);
          ctx.restore();
          break;
        }

        case 'diamond': {
          ctx.moveTo(p1.x, p1.y - dist);
          ctx.lineTo(p1.x + dist * 0.7, p1.y);
          ctx.lineTo(p1.x, p1.y + dist);
          ctx.lineTo(p1.x - dist * 0.7, p1.y);
          ctx.closePath();
          break;
        }

        case 'spiral': {
          const coils = 3;
          const maxAngle = coils * 2 * Math.PI;
          ctx.moveTo(p1.x, p1.y);
          for (let a = 0; a <= maxAngle; a += 0.1) {
            const r = (dist / maxAngle) * a;
            const x = p1.x + Math.cos(a) * r;
            const y = p1.y + Math.sin(a) * r;
            ctx.lineTo(x, y);
          }
          break;
        }
      }

      if (isFill && tool !== 'spiral') {
        ctx.fill();
      } else {
        ctx.stroke();
      }
    }

    // --- Canvas Coordinate Normalizer ---
    function getCanvasCoordinates(e) {
      const rect = mainCanvas.getBoundingClientRect();
      const scaleX = mainCanvas.width / rect.width;
      const scaleY = mainCanvas.height / rect.height;

      let clientX = e.clientX;
      let clientY = e.clientY;

      if (e.touches && e.touches.length > 0) {
        clientX = e.touches[0].clientX;
        clientY = e.touches[0].clientY;
      }

      const canvasX = (clientX - rect.left) * scaleX;
      const canvasY = (clientY - rect.top) * scaleY;

      // Update UI coordinate display
      coordDisplay.textContent = `X: ${Math.round(canvasX)} | Y: ${Math.round(canvasY)}`;

      return {
        x: clamp(canvasX, 0, 1200),
        y: clamp(canvasY, 0, 1200)
      };
    }

    // --- Pointer Interaction Event Handlers ---
    let isDrawingPending = false;

    function handlePointerStart(e) {
      e.preventDefault();
      STATE.isDrawing = true;
      const coords = getCanvasCoordinates(e);

      STATE.startPoint = coords;
      STATE.lastPoint = coords;
      STATE.currentPoint = coords;
    }

    function handlePointerMove(e) {
      if (!STATE.isDrawing) {
        getCanvasCoordinates(e); // Just update coordinate UI text
        return;
      }
      e.preventDefault();

      STATE.currentPoint = getCanvasCoordinates(e);

      // Throttled rendering via requestAnimationFrame to prevent browser freeze or DoS
      if (!isDrawingPending) {
        isDrawingPending = true;
        requestAnimationFrame(() => {
          renderDrawingFrame();
          isDrawingPending = false;
        });
      }
    }

    function handlePointerEnd(e) {
      if (!STATE.isDrawing) return;
      STATE.isDrawing = false;

      // Commit finalized stroke or shape onto main permanent canvas layer
      commitDrawingToMainCanvas();
      
      // Clear overlay preview canvas
      drawGuideOverlay();

      // Push snapshot into undo/redo history
      saveCanvasState();
    }

    function setupCanvasContext(ctx, isEraser = false) {
      ctx.lineWidth = STATE.brushSize;
      ctx.lineCap = 'round';
      ctx.lineJoin = 'round';

      if (isEraser) {
        ctx.globalCompositeOperation = 'destination-out';
        ctx.strokeStyle = 'rgba(0,0,0,1)';
        ctx.fillStyle = 'rgba(0,0,0,1)';
      } else {
        ctx.globalCompositeOperation = 'source-over';
        ctx.strokeStyle = STATE.strokeColor;
        ctx.fillStyle = STATE.strokeColor;
        ctx.globalAlpha = STATE.opacity;
      }
    }

    function renderDrawingFrame() {
      if (STATE.tool === 'pen' || STATE.tool === 'eraser') {
        // Direct freehand stroke onto main canvas
        mainCtx.save();
        setupCanvasContext(mainCtx, STATE.tool === 'eraser');

        const p1 = { x: STATE.lastPoint.x - 600, y: STATE.lastPoint.y - 600 };
        const p2 = { x: STATE.currentPoint.x - 600, y: STATE.currentPoint.y - 600 };

        executeRadialSymmetry(mainCtx, (ctx) => {
          ctx.beginPath();
          ctx.moveTo(p1.x, p1.y);
          ctx.lineTo(p2.x, p2.y);
          ctx.stroke();
        });

        mainCtx.restore();
        STATE.lastPoint = STATE.currentPoint;

      } else {
        // Live shape drag preview on guide overlay canvas
        drawGuideOverlay(); // Redraw static radial guidelines first

        guideCtx.save();
        setupCanvasContext(guideCtx, false);

        const p1 = { x: STATE.startPoint.x - 600, y: STATE.startPoint.y - 600 };
        const p2 = { x: STATE.currentPoint.x - 600, y: STATE.currentPoint.y - 600 };
        const isFill = STATE.fillMode === 'fill';

        executeRadialSymmetry(guideCtx, (ctx) => {
          renderToolShape(ctx, STATE.tool, p1, p2, isFill);
        });

        guideCtx.restore();
      }
    }

    function commitDrawingToMainCanvas() {
      if (STATE.tool === 'pen' || STATE.tool === 'eraser') return; // Pen committed incrementally

      mainCtx.save();
      setupCanvasContext(mainCtx, false);

      const p1 = { x: STATE.startPoint.x - 600, y: STATE.startPoint.y - 600 };
      const p2 = { x: STATE.currentPoint.x - 600, y: STATE.currentPoint.y - 600 };
      const isFill = STATE.fillMode === 'fill';

      executeRadialSymmetry(mainCtx, (ctx) => {
        renderToolShape(ctx, STATE.tool, p1, p2, isFill);
      });

      mainCtx.restore();
    }

    // Attach Event Listeners to Canvas (Mouse & Touch Supported)
    mainCanvas.addEventListener('mousedown', handlePointerStart);
    window.addEventListener('mousemove', handlePointerMove);
    window.addEventListener('mouseup', handlePointerEnd);

    mainCanvas.addEventListener('touchstart', handlePointerStart, { passive: false });
    window.addEventListener('touchmove', handlePointerMove, { passive: false });
    window.addEventListener('touchend', handlePointerEnd);

    // --- UI Controls Binding & Sanitization ---
    // Symmetry Axes Control
    function updateAxes(val) {
      const sanitized = clamp(val, 2, 36);
      STATE.axes = sanitized;
      sliderAxes.value = sanitized;
      numAxes.value = sanitized;
      axesValue.textContent = `${sanitized} Axes`;
      drawGuideOverlay();
    }

    sliderAxes.addEventListener('input', (e) => updateAxes(e.target.value));
    numAxes.addEventListener('change', (e) => updateAxes(e.target.value));

    // Toggles
    chkShowGuides.addEventListener('change', (e) => {
      STATE.showGuides = e.target.checked;
      drawGuideOverlay();
    });

    chkEnableRadial.addEventListener('change', (e) => {
      STATE.enableRadial = e.target.checked;
      drawGuideOverlay();
    });

    chkMirrorMode.addEventListener('change', (e) => {
      STATE.mirrorMode = e.target.checked;
      drawGuideOverlay();
    });

    // Stroke Color Picker Sync
    pickerStroke.addEventListener('input', (e) => {
      STATE.strokeColor = e.target.value;
      textStrokeHex.value = e.target.value.toUpperCase();
    });

    textStrokeHex.addEventListener('change', (e) => {
      let hex = e.target.value.trim();
      if (!hex.startsWith('#')) hex = '#' + hex;
      if (isValidHexColor(hex)) {
        STATE.strokeColor = hex;
        pickerStroke.value = hex;
      } else {
        textStrokeHex.value = STATE.strokeColor.toUpperCase();
      }
    });

    // Background Color Picker Sync
    pickerBg.addEventListener('input', (e) => {
      STATE.bgColor = e.target.value;
      textBgHex.value = e.target.value.toUpperCase();
      fillCanvasBackground(STATE.bgColor);
      drawGuideOverlay();
    });

    textBgHex.addEventListener('change', (e) => {
      let hex = e.target.value.trim();
      if (!hex.startsWith('#')) hex = '#' + hex;
      if (isValidHexColor(hex)) {
        STATE.bgColor = hex;
        pickerBg.value = hex;
        fillCanvasBackground(STATE.bgColor);
        drawGuideOverlay();
      } else {
        textBgHex.value = STATE.bgColor.toUpperCase();
      }
    });

    // Quick Color Swatches
    document.getElementById('paletteSwatches').addEventListener('click', (e) => {
      const btn = e.target.closest('button[data-color]');
      if (btn) {
        const color = btn.getAttribute('data-color');
        STATE.strokeColor = color;
        pickerStroke.value = color;
        textStrokeHex.value = color.toUpperCase();
      }
    });

    // Sliders
    sliderBrushSize.addEventListener('input', (e) => {
      const val = clamp(e.target.value, 1, 50);
      STATE.brushSize = val;
      valBrushSize.textContent = `${val} px`;
    });

    sliderOpacity.addEventListener('input', (e) => {
      const val = clamp(e.target.value, 10, 100);
      STATE.opacity = val / 100;
      valOpacity.textContent = `${val}%`;
    });

    // Tool Selection Cards
    toolButtons.forEach((btn) => {
      btn.addEventListener('click', () => {
        toolButtons.forEach((b) => b.classList.remove('active'));
        btn.classList.add('active');
        STATE.tool = btn.getAttribute('data-tool');
      });
    });

    // Fill Mode Radio Buttons
    fillModeRadios.forEach((radio) => {
      radio.addEventListener('change', (e) => {
        if (e.target.checked) STATE.fillMode = e.target.value;
      });
    });

    // --- PDF Download Export Handler ---
    btnExportPdf.addEventListener('click', () => {
      try {
        const { jsPDF } = window.jspdf;
        if (!jsPDF) {
          alert('jsPDF library failed to load. Please verify your connection.');
          return;
        }

        // Create PDF document with 1200x1200pt resolution matching canvas exactly
        const pdf = new jsPDF({
          orientation: 'portrait',
          unit: 'pt',
          format: [1200, 1200]
        });

        // Convert main drawing canvas into PNG image
        const imgData = mainCanvas.toDataURL('image/png', 1.0);

        // Render full quality image centered on PDF document page
        pdf.addImage(imgData, 'PNG', 0, 0, 1200, 1200);

        // Generate download filename with timestamp
        const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
        pdf.save(`mandala-artwork-${timestamp}.pdf`);

      } catch (err) {
        console.error('PDF Export Error:', err);
        alert('An error occurred while generating the PDF document.');
      }
    });

    // --- On Window Load Initialization ---
    window.onload = () => {
      initCanvas();
    };
  </script>
