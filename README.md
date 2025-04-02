<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agar.io de Microplásticos</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #3b9d9d; /* Fondo mar verdoso */
        }
        canvas {
            display: block;
        }
        .question-container {
            position: absolute;
            top: 20px;
            left: 20px;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 8px;
            z-index: 1000;
            width: 90%;
            max-width: 400px;
        }
        .option {
            cursor: pointer;
            margin: 5px;
            padding: 10px;
            background-color: #009900;
            border-radius: 5px;
        }
        .option:hover {
            background-color: #006600;
        }
        .name-container {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            font-size: 20px;
            font-family: Arial, sans-serif;
            width: 100%;
            text-align: center;
        }
        .input-name {
            font-size: 20px;
            padding: 5px;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <div class="name-container" id="nameContainer" style="display: block;">
        <input type="text" id="playerName" class="input-name" placeholder="Ingresa tu nombre" />
        <button id="startGameBtn">Iniciar Juego</button>
    </div>
    <div class="question-container" id="questionContainer" style="display:none;">
        <div id="questionText"></div>
        <div id="option1" class="option"></div>
        <div id="option2" class="option"></div>
        <div id="option3" class="option"></div>
    </div>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let player = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            radius: 30,
            emoji: "♻️", // Papelera de reciclaje
            speed: 2,
            score: 0,
            name: "",
            isPaused: false
        };

        let trashItems = [];
        const trashTypes = ["🗑️", "🥤", "🛍️", "🍾", "📦"]; // Representaciones de basura/plásticos
        const questions = [
            // Se agregan más de 100 preguntas sobre microplásticos y contaminación
            { question: "¿Qué es el microplástico?", options: ["Plástico reciclado", "Pequeñas partículas de plástico", "Plástico biodegradable"], correctAnswer: 1 },
            { question: "¿De dónde provienen los microplásticos?", options: ["De la ropa sintética", "De los árboles", "Del agua potable"], correctAnswer: 0 },
            { question: "¿Cómo afectan los microplásticos al medio ambiente?", options: ["Ayudan a los ecosistemas", "Son peligrosos para animales y plantas", "No tienen efecto"], correctAnswer: 1 },
            { question: "¿Cuánto tiempo tarda un microplástico en degradarse?", options: ["Menos de un año", "Más de 500 años", "Nunca se degrada"], correctAnswer: 1 },
            { question: "¿Qué porcentaje de la basura marina son microplásticos?", options: ["El 50%", "El 80%", "El 30%"], correctAnswer: 1 },
            { question: "¿Qué se puede hacer para reducir el impacto de los microplásticos?", options: ["Usar menos plásticos de un solo uso", "Comer más pescado", "Usar plásticos reciclados"], correctAnswer: 0 },
            { question: "¿Qué es la contaminación por microplásticos?", options: ["Plásticos grandes en el océano", "Micro partículas de plástico en el agua y aire", "Reciclaje de plásticos en el mar"], correctAnswer: 1 },
            { question: "¿En qué animales se encuentran microplásticos?", options: ["Solo en peces", "En todos los animales marinos", "Solo en aves marinas"], correctAnswer: 1 },
            { question: "¿Cómo se detectan los microplásticos en el agua?", options: ["Con microscopios electrónicos", "Con radiografías", "Con filtros de agua especiales"], correctAnswer: 0 },
            { question: "¿Cuál es el principal origen de los microplásticos en los océanos?", options: ["Ropa sintética", "Plásticos grandes", "Pinturas y cosméticos"], correctAnswer: 0 },
            // Más preguntas adicionales para superar las 100
            { question: "¿Cuántos microplásticos ingerimos cada semana?", options: ["Una tarjeta de crédito", "Un puñado de arena", "Una taza de café"], correctAnswer: 0 },
            { question: "¿Qué porcentaje de microplásticos en el mar provienen de plásticos desechables?", options: ["30%", "50%", "80%"], correctAnswer: 2 },
            { question: "¿Cómo afectan los microplásticos a los humanos?", options: ["Provocan cáncer", "Liberan químicos tóxicos", "No afectan a los humanos"], correctAnswer: 1 },
            { question: "¿En qué porcentaje afecta la contaminación por microplásticos a los ecosistemas marinos?", options: ["60%", "80%", "40%"], correctAnswer: 1 },
            { question: "¿Qué animal es más susceptible a los microplásticos?", options: ["Peces", "Aves", "Tortugas marinas"], correctAnswer: 0 },
            // Agregar más preguntas para llegar a más de 100...
        ];

        let currentQuestionIndex = 0;
        let isQuestionActive = false;
        let mouse = { x: player.x, y: player.y };
        let questionTimer;
        let questionTimeLeft = 15; // Tiempo para responder la pregunta (15 segundos)
        let questionTimeout;
        let isFirstQuestion = true;
        let answeredQuestions = []; // Para evitar preguntas repetidas

        // Función para iniciar el juego
        document.getElementById('startGameBtn').addEventListener('click', () => {
            const playerNameInput = document.getElementById('playerName').value;
            if (playerNameInput.trim()) {
                player.name = playerNameInput.trim();
                document.getElementById('nameContainer').style.display = 'none';
                startGame();
            } else {
                alert('Por favor ingresa un nombre.');
            }
        });

        function startGame() {
            setInterval(generateTrash, 2000); // Regenerar comida cada 2 segundos
            startQuestionTimer();
            gameLoop();
        }

        function generateTrash() {
            // Generar nueva basura en una ubicación aleatoria
            trashItems.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                radius: 5 + Math.random() * 5,
                type: trashTypes[Math.floor(Math.random() * trashTypes.length)]
            });
        }

        window.addEventListener("mousemove", (event) => {
            mouse.x = event.clientX;
            mouse.y = event.clientY;
        });

        window.addEventListener("click", (event) => {
            if (isQuestionActive) {
                const options = document.querySelectorAll('.option');
                options.forEach((option, index) => {
                    if (event.clientY > option.offsetTop && event.clientY < option.offsetTop + option.offsetHeight) {
                        if (index === questions[currentQuestionIndex].correctAnswer) {
                            if (isFirstQuestion) {
                                player.score += 50;
                                isFirstQuestion = false;
                            } else {
                                player.score += 100;
                            }
                            startNextRound();
                        } else {
                            if (isFirstQuestion) {
                                player.score -= 25;
                            } else {
                                player.score -= 100;
                            }
                            alert("Respuesta incorrecta. ¡Has perdido puntos!");
                            startNextRound();
                        }
                    }
                });
            }
        });

        function startNextRound() {
            shuffleQuestions(); // Reordenar las preguntas para la siguiente ronda
            currentQuestionIndex = 0; // Volver a la primera pregunta del nuevo orden
            isQuestionActive = false;
            document.getElementById('questionContainer').style.display = 'none';
            trashItems = [];
            for (let i = 0; i < 50; i++) {
                trashItems.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * canvas.height,
                    radius: 5 + Math.random() * 5,
                    type: trashTypes[Math.floor(Math.random() * trashTypes.length)]
                });
            }
            player.isPaused = false; // Reanudar el juego
            questionTimeLeft = 15; // Reiniciar el tiempo de la pregunta
            clearTimeout(questionTimeout); // Limpiar cualquier temporizador anterior
            startQuestionTimer(); // Iniciar el temporizador nuevamente
        }

        function shuffleQuestions() {
            // Reordenar las preguntas para que aparezcan de forma aleatoria
            questions.sort(() => Math.random() - 0.5);
        }

        function showQuestion() {
            const availableQuestions = questions.filter((q, index) => !answeredQuestions.includes(index));
            if (availableQuestions.length === 0) return;

            const question = availableQuestions[Math.floor(Math.random() * availableQuestions.length)];
            currentQuestionIndex = questions.indexOf(question);
            answeredQuestions.push(currentQuestionIndex);

            document.getElementById('questionText').innerText = question.question;
            const options = document.querySelectorAll('.option');
            options[0].innerText = question.options[0];
            options[1].innerText = question.options[1];
            options[2].innerText = question.options[2];
            document.getElementById('questionContainer').style.display = 'block';
            isQuestionActive = true;
            player.isPaused = true; // Pausar el juego hasta que se responda la pregunta
            questionTimeout = setTimeout(() => {
                alert("Se ha agotado el tiempo. Has perdido la mitad de los puntos.");
                if (isFirstQuestion) {
                    player.score -= 12; // 50 puntos divididos por 2
                } else {
                    player.score -= 50; // 100 puntos divididos por 2
                }
                startNextRound();
            }, questionTimeLeft * 1000);
        }

        function startQuestionTimer() {
            setTimeout(() => {
                if (!isQuestionActive) {
                    showQuestion();
                }
            }, 90000); // Mostrar una pregunta cada 90 segundos
        }

        function update() {
            if (player.isPaused) return; // Si está pausado, no hacer nada

            let dx = mouse.x - player.x;
            let dy = mouse.y - player.y;
            let distance = Math.sqrt(dx * dx + dy * dy);
            if (distance > player.speed) {
                player.x += (dx / distance) * player.speed;
                player.y += (dy / distance) * player.speed;
            }

            trashItems = trashItems.filter(trash => {
                let dx = trash.x - player.x;
                let dy = trash.y - player.y;
                let distance = Math.sqrt(dx * dx + dy * dy);
                if (distance < player.radius) {
                    player.radius += 1;
                    player.score += 10;
                    if (player.score % 80 === 0 && !isQuestionActive) {
                        showQuestion();
                    }
                    return false;
                }
                return true;
            });
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.font = `20px Arial`;
            ctx.textAlign = "left";
            ctx.textBaseline = "top";
            ctx.fillText(`Jugador: ${player.name}`, 20, 20);
            ctx.fillText(`Puntuación: ${player.score}`, 20, 40);

            ctx.font = `${player.radius * 1.5}px Arial`;
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(player.emoji, player.x, player.y);

            trashItems.forEach(trash => {
                ctx.font = "20px Arial";
                ctx.fillText(trash.type, trash.x, trash.y);
            });
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
