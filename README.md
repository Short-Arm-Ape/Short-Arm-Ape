<canvas id="antCanvas" style="display:block;width:100%;max-width:800px;height:400px;background:#000;margin:0 auto;"></canvas>
<script>
(function() {
    const canvas = document.getElementById('antCanvas');
    const ctx = canvas.getContext('2d');
    const CELL_SIZE = 5;

    let gridWidth, gridHeight;
    let grid = [];
    let ants = [];
    let lastTime = 0;
    let frameCount = 0;
    let fps = 0;
    let stepsPerFrame = 1;
    let stepCount = 0;

    class Ant {
        constructor(x, y, name) {
            this.name = name;
            this.initialX = x;
            this.initialY = y;
            this.reset();
        }
        reset() {
            this.x = this.initialX;
            this.y = this.initialY;
            this.direction = Math.floor(Math.random() * 4);
        }
        getColorComponents() {
            if (this.name === 'red')   return { r: 255, g: 0, b: 0 };
            if (this.name === 'green') return { r: 0, g: 255, b: 0 };
            if (this.name === 'blue')  return { r: 0, g: 0, b: 255 };
            return { r: 0, g: 0, b: 0 };
        }
        getColor() {
            if (this.name === 'red')   return '#FF0000';
            if (this.name === 'green') return '#00FF00';
            if (this.name === 'blue')  return '#0000FF';
            return '#000000';
        }
        isBlackCell(gx, gy) {
            const cell = grid[gy][gx];
            if (this.name === 'red')   return cell.r > 0;
            if (this.name === 'green') return cell.g > 0;
            if (this.name === 'blue')  return cell.b > 0;
            return false;
        }
        update() {
            const gx = Math.floor(this.x / CELL_SIZE);
            const gy = Math.floor(this.y / CELL_SIZE);
            const comp = this.getColorComponents();

            if (this.isBlackCell(gx, gy)) {
                this.direction = (this.direction + 3) % 4; // 左转
                grid[gy][gx].r = Math.max(0, grid[gy][gx].r - comp.r);
                grid[gy][gx].g = Math.max(0, grid[gy][gx].g - comp.g);
                grid[gy][gx].b = Math.max(0, grid[gy][gx].b - comp.b);
            } else {
                this.direction = (this.direction + 1) % 4; // 右转
                grid[gy][gx].r = Math.min(255, grid[gy][gx].r + comp.r);
                grid[gy][gx].g = Math.min(255, grid[gy][gx].g + comp.g);
                grid[gy][gx].b = Math.min(255, grid[gy][gx].b + comp.b);
            }

            // 移动
            switch (this.direction) {
                case 0: this.y -= CELL_SIZE; break;
                case 1: this.x += CELL_SIZE; break;
                case 2: this.y += CELL_SIZE; break;
                case 3: this.x -= CELL_SIZE; break;
            }
            // 边界环绕
            if (this.x < 0) this.x = canvas.width - CELL_SIZE;
            if (this.x >= canvas.width) this.x = 0;
            if (this.y < 0) this.y = canvas.height - CELL_SIZE;
            if (this.y >= canvas.height) this.y = 0;
        }
    }

    function initGrid() {
        grid = new Array(gridHeight);
        for (let y = 0; y < gridHeight; y++) {
            grid[y] = new Array(gridWidth);
            for (let x = 0; x < gridWidth; x++) {
                grid[y][x] = { r: 0, g: 0, b: 0 };
            }
        }
    }

    function initAnts() {
        ants = [
            new Ant(Math.floor(Math.random() * (canvas.width - CELL_SIZE)), Math.floor(Math.random() * (canvas.height - CELL_SIZE)), 'red'),
            new Ant(Math.floor(Math.random() * (canvas.width - CELL_SIZE)), Math.floor(Math.random() * (canvas.height - CELL_SIZE)), 'green'),
            new Ant(Math.floor(Math.random() * (canvas.width - CELL_SIZE)), Math.floor(Math.random() * (canvas.height - CELL_SIZE)), 'blue')
        ];
    }

    function resizeCanvas() {
        const rect = canvas.getBoundingClientRect();
        const dpr = window.devicePixelRatio || 1;
        canvas.width = Math.floor(rect.width / CELL_SIZE) * CELL_SIZE * dpr;
        canvas.height = Math.floor(rect.height / CELL_SIZE) * CELL_SIZE * dpr;
        ctx.setTransform(dpr, 0, 0, dpr, 0, 0); // 适应高分屏
        
        gridWidth = Math.floor(rect.width / CELL_SIZE);
        gridHeight = Math.floor(rect.height / CELL_SIZE);
        initGrid();
        initAnts();
    }

    function drawGrid() {
        for (let y = 0; y < gridHeight; y++) {
            for (let x = 0; x < gridWidth; x++) {
                const c = grid[y][x];
                ctx.fillStyle = `rgb(${Math.floor(c.r)},${Math.floor(c.g)},${Math.floor(c.b)})`;
                ctx.fillRect(x * CELL_SIZE, y * CELL_SIZE, CELL_SIZE, CELL_SIZE);
            }
        }
    }

    function drawAnts() {
        ants.forEach(ant => {
            ctx.fillStyle = ant.getColor();
            ctx.fillRect(ant.x, ant.y, CELL_SIZE, CELL_SIZE);
            // 方向指示
            ctx.strokeStyle = '#FFFFFF';
            ctx.lineWidth = 1;
            ctx.beginPath();
            const cx = ant.x + CELL_SIZE/2, cy = ant.y + CELL_SIZE/2;
            const hs = CELL_SIZE * 0.4;
            switch (ant.direction) {
                case 0: ctx.moveTo(cx, cy - hs); ctx.lineTo(cx - hs/2, cy + hs/2); ctx.lineTo(cx + hs/2, cy + hs/2); break;
                case 1: ctx.moveTo(cx + hs, cy); ctx.lineTo(cx - hs/2, cy - hs/2); ctx.lineTo(cx - hs/2, cy + hs/2); break;
                case 2: ctx.moveTo(cx, cy + hs); ctx.lineTo(cx - hs/2, cy - hs/2); ctx.lineTo(cx + hs/2, cy - hs/2); break;
                case 3: ctx.moveTo(cx - hs, cy); ctx.lineTo(cx + hs/2, cy - hs/2); ctx.lineTo(cx + hs/2, cy + hs/2); break;
            }
            ctx.closePath();
            ctx.stroke();
        });
    }

    function adjustSpeed() {
        const targetFps = 60;
        if (fps > targetFps + 5) {
            stepsPerFrame = Math.min(100, stepsPerFrame + 1);
        } else if (fps < targetFps - 5) {
            stepsPerFrame = Math.max(1, stepsPerFrame - 1);
        }
    }

    function animate(timestamp) {
        frameCount++;
        if (timestamp >= lastTime + 1000) {
            fps = frameCount * 1000 / (timestamp - lastTime);
            frameCount = 0;
            lastTime = timestamp;
            adjustSpeed();
        }

        ctx.fillStyle = '#000';
        ctx.fillRect(0, 0, canvas.width / (window.devicePixelRatio || 1), canvas.height / (window.devicePixelRatio || 1));
        drawGrid();

        for (let i = 0; i < stepsPerFrame; i++) {
            ants.forEach(ant => ant.update());
            stepCount++;
        }

        drawAnts();
        requestAnimationFrame(animate);
    }

    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();
    requestAnimationFrame(animate);
})();
</script>
