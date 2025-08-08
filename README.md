<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Workplace Prediction Game</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; background-color: #f4f4f9; margin: 0; padding: 20px; }
        h1 { color: #333; }
        #game-container { max-width: 700px; margin: 0 auto; }
        #bingo-card { display: grid; grid-template-columns: repeat(5, 120px); gap: 5px; margin: 20px auto; }
        .bingo-square { width: 120px; height: 120px; border: 2px solid #333; display: flex; align-items: center; justify-content: center; text-align: center; font-size: 12px; background-color: #fff; border-radius: 5px; }
        .bingo-square.selected { background-color: #aaffaa; cursor: pointer; }
        .bingo-square.free { background-color: #d3d3d3; }
        .bingo-header { font-weight: bold; background-color: #e0e0e0; }
        #player-controls, #admin-controls, #status { margin: 20px 0; }
        button { padding: 10px 20px; margin: 5px; background-color: #1976d2; color: white; border: none; border-radius: 5px; cursor: pointer; }
        button:hover { background-color: #1565c0; }
        input { padding: 8px; margin: 5px; border-radius: 5px; border: 1px solid #ccc; }
        #results { white-space: pre-wrap; font-family: monospace; text-align: left; background-color: #fff; padding: 10px; border: 1px solid #ccc; max-height: 300px; overflow-y: auto; }
        .hidden { display: none; }
    </style>
</head>
<body>
    <div id="game-container">
        <h1>Workplace Prediction Game</h1>
        <div id="player-controls">
            <p>Enter your name and select events you think will happen this week:</p>
            <input type="text" id="name-input" placeholder="Your Name">
            <button onclick="startGame()">Start Predicting</button>
            <button onclick="submitPredictions()">Submit Predictions</button>
        </div>
        <div id="admin-controls" class="hidden">
            <h2>Admin: Record Actual Events</h2>
            <button onclick="toggleAdminMode()">Enter Admin Mode</button>
            <button onclick="submitActualEvents()" class="hidden">Submit Actual Events</button>
            <button onclick="viewResults()">View Results</button>
        </div>
        <div id="bingo-card"></div>
        <div id="status"></div>
        <div id="results" class="hidden"></div>
    </div>

    <script>
        const workplaceEvents = [
            "Brendon says \"I’ll come him back\"",
            "Tbag mentions CANS per day 1-3",
            "Tbag mentions CANS per day 3-5",
            "Tbag mentions CANS per day <5",
            "George comes in and burps",
            "Phil goes to get Japanese for lunch",
            "Merredin Panel gets wrong order",
            "Joe Zito talks about kits",
            "Crocker brings up the big rig",
            "Brascin handballs something to Peter",
            "Bob codes something incorrectly",
            "Peta calls in sick",
            "Someone leaves early",
            "Brett gets incorrect order 2-3 per day",
            "Brett gets incorrect order <3 per day",
            "IT issues",
            "Someone books days off",
            "Bob makes a dirty joke",
            "Local parts delivered to wrong address",
            "Dan says \"your line is bad I can’t hear you\"",
            "John swears on the phone <5",
            "John swears on the phone >5",
            "Someone forgets a meeting",
            "The office runs out of coffee"
        ];
        let playerName = '';
        let predictions = JSON.parse(localStorage.getItem('predictions') || '{}');
        let actualEvents = JSON.parse(localStorage.getItem('actualEvents') || '[]');
        let isAdminMode = false;
        let card = [];

        function shuffle(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        function generateCard() {
            card = [];
            const shuffledEvents = shuffle([...workplaceEvents]);
            for (let i = 0; i < 25; i++) {
                if (i === 12) {
                    card.push({ text: "FREE", selected: true, isFree: true });
                } else {
                    card.push({ text: shuffledEvents[i % workplaceEvents.length], selected: false, isFree: false });
                }
            }
            renderCard();
        }

        function renderCard() {
            const cardDiv = document.getElementById('bingo-card');
            cardDiv.innerHTML = '';
            ['B', 'I', 'N', 'G', 'O'].forEach(header => {
                const headerDiv = document.createElement('div');
                headerDiv.className = 'bingo-square bingo-header';
                headerDiv.textContent = header;
                cardDiv.appendChild(headerDiv);
            });
            card.forEach((square, index) => {
                const squareDiv = document.createElement('div');
                squareDiv.className = `bingo-square ${square.selected ? 'selected' : ''} ${square.isFree ? 'free' : ''}`;
                squareDiv.textContent = square.text;
                if (!square.isFree && !isAdminMode) {
                    squareDiv.onclick = () => toggleSquare(index);
                } else if (isAdminMode && !square.isFree) {
                    squareDiv.onclick = () => toggleActualEvent(index);
                }
                cardDiv.appendChild(squareDiv);
            });
        }

        function toggleSquare(index) {
            if (!card[index].isFree) {
                card[index].selected = !card[index].selected;
                renderCard();
            }
        }

        function toggleActualEvent(index) {
            if (!card[index].isFree) {
                const eventText = card[index].text;
                if (actualEvents.includes(eventText)) {
                    actualEvents = actualEvents.filter(e => e !== eventText);
                } else {
                    actualEvents.push(eventText);
                }
                localStorage.setItem('actualEvents', JSON.stringify(actualEvents));
                renderCard();
            }
        }

        function startGame() {
            playerName = document.getElementById('name-input').value.trim();
            if (!playerName) {
                document.getElementById('status').textContent = 'Please enter a name!';
                return;
            }
            document.getElementById('player-controls').querySelector('button').style.display = 'none';
            document.getElementById('status').textContent = `Welcome, ${playerName}! Select events you think will happen.`;
            generateCard();
        }

        function submitPredictions() {
            if (!playerName) {
                document.getElementById('status').textContent = 'Please enter a name and start the game!';
                return;
            }
            const selectedEvents = card.filter(square => square.selected && !square.isFree).map(square => square.text);
            predictions[playerName] = selectedEvents;
            localStorage.setItem('predictions', JSON.stringify(predictions));
            document.getElementById('status').textContent = `Predictions submitted for ${playerName}!`;
            document.getElementById('bingo-card').innerHTML = '';
            document.getElementById('player-controls').querySelector('button').style.display = 'inline';
        }

        function toggleAdminMode() {
            const adminButton = document.getElementById('admin-controls').querySelector('button');
            if (adminButton.textContent === 'Enter Admin Mode') {
                isAdminMode = true;
                adminButton.textContent = 'Exit Admin Mode';
                document.getElementById('submit-actual-events').classList.remove('hidden');
                document.getElementById('status').textContent = 'Admin Mode: Select events that actually happened.';
                generateCard();
            } else {
                isAdminMode = false;
                adminButton.textContent = 'Enter Admin Mode';
                document.getElementById('submit-actual-events').classList.add('hidden');
                document.getElementById('status').textContent = '';
                document.getElementById('bingo-card').innerHTML = '';
            }
        }

        function submitActualEvents() {
            localStorage.setItem('actualEvents', JSON.stringify(actualEvents));
            document.getElementById('status').textContent = 'Actual events submitted!';
            renderCard();
        }

        function viewResults() {
            const resultsDiv = document.getElementById('results');
            resultsDiv.classList.remove('hidden');
            let results = '=== Prediction Results ===\n\n';
            for (const [name, preds] of Object.entries(predictions)) {
                const correct = preds.filter(event => actualEvents.includes(event)).length + (preds.includes('FREE') ? 1 : 0);
                results += `${name}: ${correct} correct predictions\n`;
                results += `Predicted: ${preds.join(', ')}\n\n`;
            }
            results += `Actual Events: ${actualEvents.join(', ')}`;
            resultsDiv.textContent = results;
        }

        // Initialize
        document.getElementById('submit-actual-events').id = 'submit-actual-events';
    </script>
</body>
</html>
