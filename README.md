<html>
    <head>
        <title>Super-Mario</title>
        <style type='text/css'>
            body {
                background: #333;
                margin: 0;
                padding: 0;
                display: flex;
                justify-content: center;
                align-items: center;
                height: 100vh;
                font-family: Arial, sans-serif;
            }
            #cenario {
                position: relative;
                background: #3ee url('ceu.png');
                width: 800px;
                height: 600px;
                overflow: hidden;
            }
            #chao {
                position: absolute;
                background: #cc3 url('chao.png');
                left: 0px;
                bottom: 0px;
                width: 800px;
                height: 72px;
            }
            #jogador {
                position: absolute;
                background: url('mario.png');
                background-size: cover;
                width: 80px;
                height: 100px;
                bottom: 65px;
                left: 150px;
            }
            #obstaculo {
                position: absolute;
                background: url('cano.png');
                background-size: cover;
                width: 160px;
                height: 130px;
                bottom: 47px;
                left: 550px;
            }
            #game-over {
                display: none;
                position: absolute;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
                font-size: 48px;
                color: red;
                font-weight: bold;
                text-shadow: 2px 2px 4px black;
            }
            #score-board {
                position: absolute;
                top: 10px;
                left: 10px;
                font-size: 24px;
                color: white;
            }
            #restart-btn {
                display: none;
                position: absolute;
                top: 60%;
                left: 50%;
                transform: translate(-50%, -50%);
                font-size: 24px;
                color: white;
                background: #000;
                padding: 10px 20px;
                cursor: pointer;
            }
            #name-input {
                display: none;
                position: absolute;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
                font-size: 24px;
                color: white;
                background: rgba(0, 0, 0, 0.8);
                padding: 20px;
                border-radius: 10px;
                text-align: center;
            }
            #name-input input {
                font-size: 20px;
                padding: 5px;
                margin-top: 10px;
            }
            #name-input button {
                font-size: 20px;
                padding: 5px 10px;
                margin-top: 10px;
                cursor: pointer;
            }
            #ranking {
                display: none;
                position: absolute;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
                font-size: 24px;
                color: white;
                background: rgba(0, 0, 0, 0.8);
                padding: 20px;
                border-radius: 10px;
                text-align: center;
            }
            #ranking h2 {
                margin: 0 0 10px 0;
            }
            #ranking ol {
                text-align: left;
                margin: 0;
                padding: 0 0 0 20px;
            }
        </style>
    </head>
    <body>
        <div id='cenario'>
            <div id='jogador'></div>
            <div id='obstaculo'></div>
            <div id='chao'></div>
            <div id='game-over'>Game Over</div>
            <div id='score-board'>Score: 0</div>
            <div id='restart-btn' onclick="restartGame()">Restart</div>
            <div id='name-input'>
                <p>Digite seu nome:</p>
                <input type='text' id='player-name' maxlength='20' />
                <button onclick="saveScore()">Salvar</button>
            </div>
            <div id='ranking'>
                <h2>Top 10 Ranking</h2>
                <ol id='ranking-list'></ol>
                <button onclick="hideRanking()">Fechar</button>
            </div>
        </div>
    </body>
    <script>
        // Variáveis do jogo
        let velocidadeDoJogo = 5;
        let pulando = false;
        let alturaInicialPlayer = 65;
        let posiçãoAtualObj = 800;
        let alturaDoPlayer = 0;
        let velocidadeDoPulo = 0;
        const gravidade = 0.5;
        const forcaDoPulo = 15;
        let jogoAtivo = true;
        let cenario = document.querySelector('#cenario');
        let jogador = document.querySelector('#jogador');
        let obstaculo = document.querySelector('#obstaculo');
        let chao = document.querySelector('#chao');
        let gameOver = document.querySelector('#game-over');
        let scoreBoard = document.querySelector('#score-board');
        let restartBtn = document.querySelector('#restart-btn');
        let nameInput = document.querySelector('#name-input');
        let ranking = document.querySelector('#ranking');
        let rankingList = document.querySelector('#ranking-list');
        let score = 0;

        // Função para iniciar o jogo
        function inicio() {
            atualiza();
        }

        // Loop do jogo
        function atualiza() {
            if (!jogoAtivo) return;

            posiçãoAtualObj -= velocidadeDoJogo;
            movimentaPlayer(jogador);
            movimenta(obstaculo);
            aplicaGravidade();
            verificaColisao();
            atualizaScore();
            requestAnimationFrame(atualiza);
        }

        // Movimenta o obstáculo
        function movimenta(objeto) {
            objeto.style.left = posiçãoAtualObj + 'px';
            if (posiçãoAtualObj < -160) {
                posiçãoAtualObj = 800;
                velocidadeDoJogo += 0.5;
                score += 1;
            }
        }

        // Movimenta o jogador
        function movimentaPlayer(objeto) {
            objeto.style.bottom = (alturaInicialPlayer + alturaDoPlayer) + 'px';
        }

        // Aplica gravidade
        function aplicaGravidade() {
            if (pulando) {
                alturaDoPlayer += velocidadeDoPulo;
                velocidadeDoPulo -= gravidade;
                if (alturaDoPlayer <= 0) {
                    alturaDoPlayer = 0;
                    pulando = false;
                    velocidadeDoPulo = 0;
                }
            }
        }

        // Verifica colisão
        function verificaColisao() {
            let jogadorRect = jogador.getBoundingClientRect();
            let obstaculoRect = obstaculo.getBoundingClientRect();

            jogadorRect = {
                left: jogadorRect.left + 30,
                right: jogadorRect.right - 30,
                top: jogadorRect.top + 20,
                bottom: jogadorRect.bottom - 10
            };

            obstaculoRect = {
                left: obstaculoRect.left + 40,
                right: obstaculoRect.right - 40,
                top: obstaculoRect.top + 20,
                bottom: obstaculoRect.bottom - 10
            };

            if (
                jogadorRect.left < obstaculoRect.right &&
                jogadorRect.right > obstaculoRect.left &&
                jogadorRect.bottom > obstaculoRect.top &&
                jogadorRect.top < obstaculoRect.bottom
            ) {
                gameOver.style.display = 'block';
                restartBtn.style.display = 'block';
                nameInput.style.display = 'block';
                jogoAtivo = false;
            }
        }

        // Atualiza o score
        function atualizaScore() {
            scoreBoard.innerText = `Score: ${score}`;
        }

        // Reinicia o jogo
        function restartGame() {
            jogoAtivo = true;
            gameOver.style.display = 'none';
            restartBtn.style.display = 'none';
            nameInput.style.display = 'none';
            ranking.style.display = 'none';
            posiçãoAtualObj = 800;
            alturaDoPlayer = 0;
            velocidadeDoJogo = 5;
            score = 0;
            inicio();
        }

        // Salva o score no ranking
        function saveScore() {
            const playerName = document.querySelector('#player-name').value.trim();
            if (playerName === '') {
                alert('Por favor, insira seu nome!');
                return;
            }

            const playerData = { name: playerName, score: score };
            let rankingData = JSON.parse(localStorage.getItem('ranking')) || [];
            rankingData.push(playerData);
            rankingData.sort((a, b) => b.score - a.score); // Ordena por score (maior para menor)
            rankingData = rankingData.slice(0, 10); // Mantém apenas os top 10
            localStorage.setItem('ranking', JSON.stringify(rankingData));

            nameInput.style.display = 'none';
            showRanking();
        }

        // Exibe o ranking
        function showRanking() {
            const rankingData = JSON.parse(localStorage.getItem('ranking')) || [];
            rankingList.innerHTML = rankingData
                .map((player, index) => `<li>${index + 1}. ${player.name} - ${player.score}</li>`)
                .join('');
            ranking.style.display = 'block';
        }

        // Fecha o ranking
        function hideRanking() {
            ranking.style.display = 'none';
        }

        // Evento de pulo
        window.addEventListener('keydown', function (evento) {
            if (evento.code == 'Space' && !pulando && jogoAtivo) {
                pulando = true;
                velocidadeDoPulo = forcaDoPulo;
            }
        });

        // Inicia o jogo
        inicio();
    </script>
</html>
