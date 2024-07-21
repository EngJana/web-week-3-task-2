# web-week-3-task-2
smart methods internship week 3 web desapline 

## Voice to Text Converter

This project converts spoken words into text using the Web Speech API and stores the transcriptions in a MySQL database. The application includes a simple front-end interface for recording speech and a back-end PHP script for saving the transcribed text to a database.

### Prerequisites

- A web server with PHP support (xampp is used here)
- MySQL database
- Modern web browser 

### Setup Instructions

#### 1. Database Configuration

1. **Create Database and Table:**

   Run the following SQL commands to create the necessary database and table:

   ```sql
   CREATE DATABASE voice_to_text;
   USE voice_to_text;
   CREATE TABLE transcriptions (
       id INT AUTO_INCREMENT PRIMARY KEY,
       transcribed_text TEXT NOT NULL,
       language VARCHAR(5) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

2. **Update Database Connection Settings:**

   Modify the following lines in `save_text.php` to match your database credentials:

   ```php
   $servername = "localhost";
   $username = "root";
   $password = "";
   $dbname = "voice_to_text";
   ```

#### 2. Front-End: `index.html`

Create an `index.html` file with the following content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice to Text</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
        }
        #textArea {
            width: 100%;
            height: 100px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Voice to Text Converter</h1>

    <!-- Language Selection -->
    <label for="language">Choose Language:</label>
    <select id="language">
        <option value="en-US">English</option>
        <option value="ar-SA">Arabic</option>
    </select>

    <br><br>

    <button id="startButton">Start Recording</button>
    <textarea id="textArea" placeholder="Your transcribed text will appear here..."></textarea>
    <form method="POST" action="save_text.php">
        <input type="hidden" name="transcribed_text" id="transcribed_text">
        <input type="hidden" name="language" id="language_input">
        <button type="submit">Save to Database</button>
    </form>

    <script>
        let recognition;
        const startButton = document.getElementById('startButton');
        const textArea = document.getElementById('textArea');
        const transcribedText = document.getElementById('transcribed_text');
        const languageSelect = document.getElementById('language');
        const languageInput = document.getElementById('language_input');

        function startRecording() {
            if (!('SpeechRecognition' in window) && !('webkitSpeechRecognition' in window)) {
                alert("Your browser does not support speech recognition. Please use Google Chrome.");
                return;
            }

            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.lang = languageSelect.value; // Set language based on selection
            recognition.continuous = false;
            recognition.interimResults = false;

            recognition.onstart = function() {
                console.log("Recording started...");
                startButton.textContent = 'Recording...';
                startButton.disabled = true;
            };

            recognition.onresult = function(event) {
                const speechResult = event.results[0][0].transcript;
                textArea.value = speechResult;
                transcribedText.value = speechResult;
                languageInput.value = languageSelect.value; // Set the language input value
            };

            recognition.onerror = function(event) {
                console.error("Error occurred in recognition: " + event.error);
                alert("Error occurred in recognition: " + event.error);
            };

            recognition.onend = function() {
                console.log("Recording ended.");
                startButton.textContent = 'Start Recording';
                startButton.disabled = false;
            };

            recognition.start();
        }

        startButton.addEventListener('click', startRecording);
    </script>
</body>
</html>
```

#### Explanation of `index.html`:

1. **HTML Structure:**
   - `<!DOCTYPE html>`: Declares the document type as HTML5.
   - `<html lang="en">`: Sets the language of the document to English.

2. **Head Section:**
   - `<meta charset="UTF-8">`: Sets the character encoding to UTF-8.
   - `<meta name="viewport" content="width=device-width, initial-scale=1.0">`: Ensures the page is responsive on different devices.
   - `<title>Voice to Text</title>`: Sets the title of the page.
   - `<style>`: Contains CSS to style the page.

3. **Body Section:**
   - `<h1>Voice to Text Converter</h1>`: A heading for the application.
   - `<label for="language">Choose Language:</label>`: A label for the language selection dropdown.
   - `<select id="language">`: A dropdown menu to select the language.
   - `<button id="startButton">Start Recording</button>`: A button to start the recording.
   - `<textarea id="textArea" placeholder="Your transcribed text will appear here..."></textarea>`: A textarea to display the transcribed text.
   - `<form method="POST" action="save_text.php">`: A form to submit the transcribed text to the server.
   - `<input type="hidden" name="transcribed_text" id="transcribed_text">`: A hidden input to store the transcribed text.
   - `<input type="hidden" name="language" id="language_input">`: A hidden input to store the selected language.
   - `<button type="submit">Save to Database</button>`: A button to submit the form.

4. **JavaScript Section:**
   - Defines and initializes variables for the elements.
   - `startRecording()` function handles the speech recognition process.
   - Event listeners handle the start, result, error, and end events of the speech recognition.

#### 3. Back-End: `save_text.php`

Create a `save_text.php` file with the following content:

```php
<?php
// Database connection settings
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "voice_to_text";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Get the transcribed text and language from the POST request
if (isset($_POST['transcribed_text']) && isset($_POST['language'])) {
    $transcribed_text = $_POST['transcribed_text'];
    $language = $_POST['language'];

    // Prepare and bind
    $stmt = $conn->prepare("INSERT INTO transcriptions (transcribed_text, language) VALUES (?, ?)");
    if ($stmt === false) {
        die("Prepare failed: (" . $conn->errno . ") " . $conn->error);
    }

    $stmt->bind_param("ss", $transcribed_text, $language);

    // Execute the statement
    if ($stmt->execute()) {
        // Redirect to the first page after successful insertion
        header("Location: index.html?success=1");
        exit;
    } else {
        echo "Error: " . $stmt->error;
    }

    // Close connections
    $stmt->close();
} else {
    echo "No transcribed text or language provided.";
}

$conn->close();
?>
```

#### Explanation of `save_text.php`:

1. **Database Connection:**
   - `$servername`, `$username`, `$password`, `$dbname`: Variables to store database connection details.
   - `new mysqli(...)`: Creates a new database connection.
   - `if ($conn->connect_error)`: Checks if the connection failed and exits if true.

2. **Handling POST Request:**
   - `if (isset($_POST['transcribed_text']) && isset($_POST['language']))`: Checks if the necessary POST parameters are set.
   - `$transcribed_text`, `$language`: Variables to store the transcribed text and language from the POST request.

3. **Prepare and Bind:**
   - `prepare(...)`: Prepares an SQL statement for execution.
   - `if ($stmt === false)`: Checks if the preparation failed and exits if true.
   - `bind_param("ss", ...)`: Binds parameters to the SQL statement.

4. **Execute and Close:**
   - `if ($stmt->execute())`: Executes the prepared statement and checks if it was successful.
   - `header("Location: index.html?success=1")`: Redirects to the index page after successful insertion.
   - `exit`: Stops the script execution after redirection.
   - `echo "Error: " . $stmt->error`: Outputs an error message if the execution failed.
   - `$stmt->close()`: Closes the prepared statement.
   - `$conn->close()`: Closes the database connection.

### Usage Instructions

1. Open `index.html` in a web browser.
2. Select the desired language from the drop-down menu.
3. Click the "Start Recording" button and begin speaking.
4. The transcribed text will appear in the text area.
5. Click the "Save to Database" button to store the transcription in the database.

<img width="959" alt="web page 2" src="https://github.com/user-attachments/assets/f4f096d5-2fd7-41cd-bf9a-7aa597592522">

<img width="810" alt="dataset" src="https://github.com/user-attachments/assets/620c6ed6-8fa5-4167-bbc5-88fcc92b9589">


