# Game-a-joystick.
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { margin: 0; overflow: hidden; background: #f0f0f0; touch-action: none; }
        #gameCanvas { background: #fff; display: block; margin: auto; }
        .joystick-container {
            position: absolute; bottom: 80px; left: 20px; width: 120px; height: 120px;
            background: rgba(0, 0, 0, 0.3); border-radius: 50%; display: flex; justify-content: center; align-items: center;
        }
        .joystick { width: 50px; height: 50px; background: gray; border-radius: 50%; position: absolute; }
        .fps { position: absolute; top: 10px; left: 10px; color: black; font-size: 18px; }
        .stats {
            position: absolute;
            top: 10px;
            right: 10px;
            color: black;
            font-size: 16px;
            text-align: right;
            background: rgba(255, 255, 255, 0.7);
            padding: 5px;
            border-radius: 5px;
            max-width: 250px;
            max-height: 200px;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <div class="joystick-container"><div class="joystick" id="stick"></div></div>
    <div class="fps" id="fps">FPS: 0</div>
    <div class="stats" id="stats">Loading...</div>

    <script>
        const c = document.getElementById('gameCanvas'), ctx = c.getContext('2d'),
              f = document.getElementById('fps'), statsEl = document.getElementById('stats');
        let p = { x: 200, y: 200, w: 30, h: 30, dx: 0, dy: 0 }, 
            j = document.getElementById('stick'), jA = false, sX, sY, lT = 0, 
            maxSpeed = 0, minSpeed = Infinity, pathLength = 0, lastTouchTime = 0;
        const s = 1.5, safeZone = { x: c.width - 180, y: 0, w: 180, h: 140 };
        c.width = innerWidth; c.height = innerHeight;

        const lang = navigator.language || navigator.userLanguage;
        let language = "en";

        const translations = {
            "en": {
                "fps": "FPS",
                "x": "X",
                "y": "Y",
                "dx": "DX",
                "dy": "DY",
                "width": "Width",
                "height": "Height",
                "scale": "Scale",
                "joystick": "Joystick",
                "maxSpeed": "Max Speed",
                "minSpeed": "Min Speed",
                "pathLength": "Path Length",
                "lastTouch": "Last Touch"
            },
            "ru": {
                "fps": "FPS",
                "x": "X",
                "y": "Y",
                "dx": "DX",
                "dy": "DY",
                "width": "Ширина",
                "height": "Высота",
                "scale": "Масштаб",
                "joystick": "Джойстик",
                "maxSpeed": "Макс. скорость",
                "minSpeed": "Мин. скорость",
                "pathLength": "Длина пути",
                "lastTouch": "Последнее касание"
            },
            "es": {
                "fps": "FPS",
                "x": "X",
                "y": "Y",
                "dx": "DX",
                "dy": "DY",
                "width": "Ancho",
                "height": "Altura",
                "scale": "Escala",
                "joystick": "Joystick",
                "maxSpeed": "Velocidad máxima",
                "minSpeed": "Velocidad mínima",
                "pathLength": "Longitud del camino",
                "lastTouch": "Último toque"
            },
            "de": {
                "fps": "FPS",
                "x": "X",
                "y": "Y",
                "dx": "DX",
                "dy": "DY",
                "width": "Breite",
                "height": "Höhe",
                "scale": "Skala",
                "joystick": "Joystick",
                "maxSpeed": "Maximale Geschwindigkeit",
                "minSpeed": "Minimale Geschwindigkeit",
                "pathLength": "Weglänge",
                "lastTouch": "Letzter Touch"
            },
            "fr": {
                "fps": "FPS",
                "x": "X",
                "y": "Y",
                "dx": "DX",
                "dy": "DY",
                "width": "Largeur",
                "height": "Hauteur",
                "scale": "Échelle",
                "joystick": "Joystick",
                "maxSpeed": "Vitesse max",
                "minSpeed": "Vitesse min",
                "pathLength": "Longueur du chemin",
                "lastTouch": "Dernier toucher"
            },
        };

        if (lang.includes("ru") || lang.includes("uk") || lang.includes("by") || lang.includes("kz")) {
            language = "ru";
        } else if (lang.includes("es")) {
            language = "es";
        } else if (lang.includes("de")) {
            language = "de";
        } else if (lang.includes("fr")) {
            language = "fr";
        } else if (lang.includes("en")) {
            language = "en";
        }

        j.addEventListener('touchstart', e => {
            jA = true;
            [sX, sY] = [e.touches[0].clientX, e.touches[0].clientY];
            lastTouchTime = (performance.now() / 1000).toFixed(1);
        });

        document.addEventListener('touchmove', e => {
            if (!jA) return;
            let [dx, dy] = [e.touches[0].clientX - sX, e.touches[0].clientY - sY];
            j.style.transform = `translate(${dx = Math.max(-40, Math.min(40, dx))}px, ${dy = Math.max(-40, Math.min(40, dy))}px)`;
            [p.dx, p.dy] = [dx / 20, dy / 20];

            let speed = Math.sqrt(p.dx ** 2 + p.dy ** 2);
            maxSpeed = Math.max(maxSpeed, speed);
            minSpeed = Math.min(minSpeed, speed);
        });

        document.addEventListener('touchend', () => {
            jA = false;
            j.style.transform = 'translate(0, 0)';
            p.dx = p.dy = 0;
        });

        function loop(t) {
            if (lT) f.textContent = `${translations[language].fps}: ${Math.round(1000 / (t - lT))}`;
            lT = t;

            let prevX = p.x, prevY = p.y;
            let newX = Math.max(0, Math.min(c.width - p.w, p.x + p.dx * s));
            let newY = Math.max(0, Math.min(c.height - p.h - 150, p.y + p.dy * s));

            if (!(newX + p.w > safeZone.x && newY < safeZone.y + safeZone.h)) {
                p.x = newX;
                p.y = newY;
            }

            pathLength += Math.sqrt((p.x - prevX) ** 2 + (p.y - prevY) ** 2);

            ctx.clearRect(0, 0, c.width, c.height);
            ctx.fillStyle = 'blue';
            ctx.fillRect(p.x, p.y, p.w, p.h);

            statsEl.innerHTML = `
                ${translations[language].x}: ${p.x.toFixed(1)}, ${translations[language].y}: ${p.y.toFixed(1)} <br>
                ${translations[language].dx}: ${p.dx.toFixed(2)}, ${translations[language].dy}: ${p.dy.toFixed(2)} <br>
                ${translations[language].width}: ${p.w}, ${translations[language].height}: ${p.h} <br>
                ${translations[language].scale}: ${s} <br>
                ${translations[language].joystick}: ${jA ? 'Active' : 'Inactive'} <br>
                ${translations[language].maxSpeed}: ${maxSpeed.toFixed(2)} <br>
                ${translations[language].minSpeed}: ${minSpeed === Infinity ? 0 : minSpeed.toFixed(2)} <br>
                ${translations[language].pathLength}: ${pathLength.toFixed(1)} <br>
                ${translations[language].lastTouch}: ${lastTouchTime}s
            `;

            requestAnimationFrame(loop);
        }
        loop(0);
    </script>
</body>
</html>
