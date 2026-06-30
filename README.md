<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 20px; }
        img { max-width: 300px; border: 1px solid #ccc; }
        button { display: block; margin: 10px auto; padding: 10px 20px; cursor: pointer; }
        #score-board { font-size: 1.2em; font-weight: bold; margin-bottom: 10px; }
        #student-entry { margin-bottom: 15px; }
        #student-name { padding: 8px; width: 240px; max-width: 100%; }
        #submission-controls { margin-top: 15px; }
        #submission-controls button { width: 160px; }
        #history-container { margin-top: 25px; padding: 15px; border: 1px solid #ccc; border-radius: 6px; max-width: 420px; margin-left: auto; margin-right: auto; text-align: left; }
        #history-container h3 { margin-top: 0; }
        #history-list { list-style: none; padding-left: 0; margin: 0; }
        #history-list li { margin-bottom: 8px; }
    </style>
</head>
<body>

    <div id="quiz-container">
        <h2>Identify the Fingering</h2>
        <div id="student-entry">
            <label for="student-name">Your Name:</label>
            <input id="student-name" type="text" placeholder="Enter your name">
        </div>
        <div id="score-board">Score: 0</div>
        <img id="fingering-img" src="" alt="Saxophone Fingering">
        
        <div id="options">
            <button onclick="checkAnswer(1)"></button>
            <button onclick="checkAnswer(2)"></button>
            <button onclick="checkAnswer(3)"></button>
            <button onclick="checkAnswer(4)"></button>
        </div>
        <p id="result"></p>
        <button id="next-button" onclick="nextQuestion()">Next Question</button>
        <div id="submission-controls">
            <button id="submit-button" onclick="submitResult()" disabled>Submit Score</button>
            <button id="copy-button" onclick="copyResult()" disabled>Copy Result</button>
            <button id="google-button" onclick="submitToGoogleForm()" disabled>Submit to Google Form</button>
            <p style="font-size:0.9em; color:#555; max-width:420px; margin:0 auto;">Set your Google Form to restrict responses to the sjjtitans.org domain and share the sheet only with nvanvorhis@sjjtitans.org.</p>
        </div>
        <iframe name="google-form-iframe" style="display:none;"></iframe>
        <div id="history-container">
            <h3>Your Recent Progress</h3>
            <p id="history-note">Enter your name to load saved history for this browser.</p>
            <ul id="history-list"></ul>
        </div>
    </div>

    <script>
    const questions = [
        { img: "Saxophone G4.gif", options: ["B flat", "G", "F sharp", "D"], correct: 2 },
        { img: "Saxophone D4.gif", options: ["A", "C", "D", "E"], correct: 3 },
        { img: "Saxophone Eb5.gif", options: ["D", "E flat", "F", "G"], correct: 2 },
        { img: "Saxophone A4.gif", options: ["A", "B", "F", "D"], correct: 1 },
        { img: "Saxophone F4.gif", options: ["G", "F", "A", "B"], correct: 2 },
        { img: "Saxophone E4.gif", options: ["B", "F", "G", "E"], correct: 4 },
        { img: "Saxophone Bb 4.gif", options: ["B flat", "C", "D", "E"], correct: 1 },
        { img: "Saxophone C5.gif", options: ["E", "D", "C", "F"], correct: 3 }
    ];

    const maxQuestions = 12;
    const storageKey = 'saxophoneQuizHistory';
    let currentIndex = 0;
    let score = 0;
    let answeredCorrectly = false; // Prevents double-scoring
    let totalQuestions = 0;

    function shuffle(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
    }

    function updateScoreBoard() {
        document.getElementById('score-board').innerText = `Score: ${score} (${currentIndex + 1}/${totalQuestions})`;
    }

    function getHistory() {
        const raw = localStorage.getItem(storageKey);
        return raw ? JSON.parse(raw) : [];
    }

    function saveHistory(history) {
        localStorage.setItem(storageKey, JSON.stringify(history));
    }

    function addSubmission(name) {
        const history = getHistory();
        history.push({
            name,
            score,
            total: totalQuestions,
            timestamp: new Date().toISOString()
        });
        saveHistory(history);
    }

    function getStudentHistory(name) {
        if (!name) return [];
        return getHistory().filter(entry => entry.name.toLowerCase() === name.toLowerCase()).slice(-10).reverse();
    }

    function formatDate(timestamp) {
        const date = new Date(timestamp);
        return date.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }) + ' ' + date.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' });
    }

    function updateHistoryDisplay(name) {
        const note = document.getElementById('history-note');
        const list = document.getElementById('history-list');
        list.innerHTML = '';

        if (!name) {
            note.textContent = 'Enter your name to load saved history for this browser.';
            return;
        }

        const history = getStudentHistory(name);
        if (history.length === 0) {
            note.textContent = `No previous history yet for ${name}.`;
            return;
        }

        note.textContent = `Recent progress for ${name}:`;
        history.forEach(entry => {
            const item = document.createElement('li');
            item.textContent = `${formatDate(entry.timestamp)} — ${entry.score}/${entry.total}`;
            list.appendChild(item);
        });
    }

    const googleFormConfig = {
        actionUrl: 'https://docs.google.com/forms/d/e/1FAIpQLSfQwdwKB6DktXjfuq7tJrH-VczXXrtXtJ_SexdHCSekLI4_NA/formResponse',
        nameField: 'entry.0000000000',
        scoreField: 'entry.0000000000',
        dateField: 'entry.0000000000'
    };

    function updateSubmissionControls() {
        const submit = document.getElementById('submit-button');
        const copy = document.getElementById('copy-button');
        const google = document.getElementById('google-button');
        const completed = currentIndex >= totalQuestions;
        submit.disabled = !completed;
        copy.disabled = !completed;
        google.disabled = !completed;
    }

    function submitToGoogleForm() {
        const name = document.getElementById('student-name').value.trim();
        if (!name) {
            alert('Please enter your name before submitting.');
            return;
        }

        if (currentIndex < totalQuestions) {
            alert('Please finish the quiz before submitting.');
            return;
        }

        const dateText = formatDate(new Date().toISOString());
        const form = document.createElement('form');
        form.method = 'POST';
        form.action = googleFormConfig.actionUrl;
        form.target = 'google-form-iframe';
        form.style.display = 'none';

        const inputs = {
            [googleFormConfig.nameField]: name,
            [googleFormConfig.scoreField]: `${score}/${totalQuestions}`,
            [googleFormConfig.dateField]: dateText
        };

        for (const [key, value] of Object.entries(inputs)) {
            const input = document.createElement('input');
            input.type = 'hidden';
            input.name = key;
            input.value = value;
            form.appendChild(input);
        }

        document.body.appendChild(form);
        form.submit();
        document.body.removeChild(form);

        document.getElementById('result').innerText = `Submitted to Google Form! Name: ${name} | Score: ${score}/${totalQuestions} | Date: ${dateText}`;
        document.getElementById('result').style.color = 'darkgreen';
    }

    function endQuiz() {
        document.getElementById('result').innerText = `Quiz complete! Final score: ${score}/${totalQuestions}`;
        document.getElementById('result').style.color = "blue";
        document.getElementById('next-button').disabled = true;
        const buttons = document.getElementById('options').getElementsByTagName('button');
        for (let i = 0; i < buttons.length; i++) {
            buttons[i].disabled = true;
        }
        updateSubmissionControls();
    }

    function loadQuestion() {
        if (currentIndex >= totalQuestions) {
            endQuiz();
            return;
        }

        const q = questions[currentIndex];
        document.getElementById('fingering-img').src = q.img;
        document.getElementById('result').innerText = "";
        answeredCorrectly = false; // Reset for new question

        updateScoreBoard();

        const buttons = document.getElementById('options').getElementsByTagName('button');
        for (let i = 0; i < buttons.length; i++) {
            buttons[i].innerText = q.options[i];
            buttons[i].disabled = false;
            buttons[i].onclick = () => checkAnswer(i + 1);
        }
    }

    function checkAnswer(choice) {
        const res = document.getElementById('result');
        if(choice === questions[currentIndex].correct) {
            if (!answeredCorrectly) {
                score++;
                answeredCorrectly = true;
            }
            res.innerText = "Correct! Great job.";
            res.style.color = "green";
        } else {
            res.innerText = "Not quite, try again!";
            res.style.color = "red";
        }
        updateScoreBoard();
    }

    function nextQuestion() {
        currentIndex++;
        loadQuestion();
    }

    function submitResult() {
        const name = document.getElementById('student-name').value.trim();
        if (!name) {
            alert('Please enter your name before submitting.');
            return;
        }

        if (currentIndex < totalQuestions) {
            alert('Please finish the quiz before submitting.');
            return;
        }

        addSubmission(name);
        updateHistoryDisplay(name);

        const dateText = formatDate(new Date().toISOString());
        const message = `Name: ${name}\nScore: ${score}/${totalQuestions}\nDate: ${dateText}`;
        document.getElementById('result').innerText = `Submitted! ${message}`;
        document.getElementById('result').style.color = 'purple';
    }

    function copyResult() {
        const name = document.getElementById('student-name').value.trim();
        const dateText = formatDate(new Date().toISOString());
        const message = name
            ? `Name: ${name}\nScore: ${score}/${totalQuestions}\nDate: ${dateText}`
            : `Score: ${score}/${totalQuestions}\nDate: ${dateText}`;
        navigator.clipboard.writeText(message).then(() => {
            alert('Result copied to clipboard.');
        }, () => {
            alert('Could not copy the result.');
        });
    }

    document.getElementById('student-name').addEventListener('input', event => {
        updateHistoryDisplay(event.target.value.trim());
    });

    // Initialize
    shuffle(questions);
    if (questions.length > maxQuestions) {
        questions.length = maxQuestions;
    }
    totalQuestions = questions.length;
    loadQuestion();
    updateSubmissionControls();
</script>
</body>
</html>
