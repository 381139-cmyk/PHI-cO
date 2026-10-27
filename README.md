// Check Victory / Level Complete
            if (levelCleared && gameState === 'PLAYING') {
                gameState = 'VICTORY';
                victoryScreen.classList.remove('hidden');
                victoryTitle.textContent = `第 ${level} 關完成！`;
                victoryDesc.textContent = `得分：${score} 分！準備好進入更高難度的挑戰了嗎？`;
            }
        }

        // Update Power-ups
        for (let i = powerUps.length - 1; i >= 0; i--) {
            let p = powerUps[i];
            p.y += p.dy;

            // Paddle catches Power-up
            if (
                p.y + p.height / 2 >= paddle.y &&
                p.y - p.height / 2 <= paddle.y + paddle.height &&
                p.x >= paddle.x &&
                p.x <= paddle.x + paddle.width
            ) {
                playSound('powerup');
                if (p.type === 'expand') {
                    paddle.width = Math.min(paddle.width + 40, 220);
                    showToast("獲得【寬板】特效！", "fa-arrows-split-up-and-left");
                } else if (p.type === 'multiball') {
                    const baseBall = balls[0] || createBall();
                    balls.push(createBall(baseBall.x, baseBall.y, -3.5, -4.5));
                    balls.push(createBall(baseBall.x, baseBall.y, 3.5, -4.5));
                    showToast("獲得【多球】支援！", "fa-circle-dot");
                } else if (p.type === 'life') {
                    lives++;
                    updateHUD();
                    showToast("獲得【加命】補給！", "fa-heart");
                }
                createParticles(p.x, p.y, p.color);
                powerUps.splice(i, 1);
                continue;
            }

            // Off screen removal
            if (p.y > VIRTUAL_HEIGHT) {
                powerUps.splice(i, 1);
            }
        }

        // Update Particles
        for (let i = particles.length - 1; i >= 0; i--) {
            let p = particles[i];
            p.x += p.dx;
            p.y += p.dy;
            p.alpha -= p.decay;
            if (p.alpha <= 0) {
                particles.splice(i, 1);
            }
        }
    }

    // --- Render / Draw Functions ---
    function drawPaddle() {
        ctx.save();
        ctx.shadowBlur = 15;
        ctx.shadowColor = '#3b82f6';
        ctx.fillStyle = paddle.color;
        
        // Rounded rectangle for neon paddle
        const radius = 8;
        ctx.beginPath();
        ctx.roundRect(paddle.x, paddle.y, paddle.width, paddle.height, radius);
        ctx.fill();

        // Inner gloss line
        ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
        ctx.beginPath();
        ctx.roundRect(paddle.x + 4, paddle.y + 2, paddle.width - 8, 3, 2);
        ctx.fill();
        ctx.restore();
    }

    function drawBalls() {
        balls.forEach(ball => {
            ctx.save();
            ctx.shadowBlur = 12;
            ctx.shadowColor = '#60a5fa';
            ctx.fillStyle = ball.color;
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fill();

            // Ball inner shine
            ctx.fillStyle = '#ffffff';
            ctx.beginPath();
            ctx.arc(ball.x - 2, ball.y - 2, ball.radius * 0.35, 0, Math.PI * 2);
            ctx.fill();
            ctx.restore();
        });
    }

    function drawBricks() {
        for (let c = 0; c < brickColumnCount; c++) {
            for (let r = 0; r < brickRowCount; r++) {
                let b = bricks[c][r];
                if (b.status === 1) {
                    ctx.save();
                    ctx.shadowBlur = 8;
                    ctx.shadowColor = b.color.fill;
                    ctx.fillStyle = b.color.fill;
                    ctx.strokeStyle = b.color.border;
                    ctx.lineWidth = 2;

                    ctx.beginPath();
                    ctx.roundRect(b.x, b.y, brickWidth, brickHeight, 5);
                    ctx.fill();
                    ctx.stroke();

                    // Inner border effect for neon grid look
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
                    ctx.lineWidth = 1;
                    ctx.strokeRect(b.x + 3, b.y + 3, brickWidth - 6, brickHeight - 6);

                    ctx.restore();
                }
            }
        }
    }

    function drawPowerUps() {
        powerUps.forEach(p => {
            ctx.save();
            ctx.shadowBlur = 10;
            ctx.shadowColor = p.color;
            ctx.fillStyle = p.color;
            
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.width / 2, 0, Math.PI * 2);
            ctx.fill();

            // Draw simple symbol inside power-up
            ctx.fillStyle = '#ffffff';
            ctx.font = '10px fontawesome, sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(p.type === 'expand' ? '↔' : p.type === 'multiball' ? '●' : '+', p.x, p.y);

            ctx.restore();
        });
    }

    function drawParticles() {
        particles.forEach(p => {
            ctx.save();
            ctx.globalAlpha = Math.max(0, p.alpha);
            ctx.fillStyle = p.color;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.restore();
        });
    }

    function render() {
        // Clear canvas background
        ctx.clearRect(0, 0, VIRTUAL_WIDTH, VIRTUAL_HEIGHT);

        drawBricks();
        drawPowerUps();
        drawParticles();
        drawPaddle();
        drawBalls();
    }

    // --- Main Game Loop ---
    function gameLoop() {
        update();
        render();
        requestAnimationFrame(gameLoop);
    }

    // --- Info Modal Helpers ---
    function openInfoModal() {
        document.getElementById('infoModal').classList.remove('hidden');
    }

    function closeInfoModal() {
        document.getElementById('infoModal').classList.add('hidden');
    }

    // Start game loop engine
    requestAnimationFrame(gameLoop);
</script>
