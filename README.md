

### 2. `app.py` (Backend Logic)
```python
from flask import Flask, render_template, request, redirect, url_for, session, send_from_directory
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.secret_key = 'your_secret_key_here'  # Replace with a real secret key
UPLOAD_FOLDER = 'uploads'
ALLOWED_EXTENSIONS = {'mp4', 'webm', 'ogg', 'mov'}

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Dummy user data (for demonstration)
users = {
    "admin": "password123"
}

# Utility function to check allowed file extensions
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

# Route for home page displaying videos
@app.route('/')
def index():
    # List all videos in uploads
    videos = os.listdir(app.config['UPLOAD_FOLDER'])
    videos = [video for video in videos if allowed_file(video)]
    return render_template('index.html', videos=videos, logged_in='username' in session)

# About page
@app.route('/about')
def about():
    return render_template('about.html', logged_in='username' in session)

# Contact us page
@app.route('/contact')
def contact():
    return render_template('contact.html', logged_in='username' in session)

# Login page
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username in users and users[username] == password:
            session['username'] = username
            return redirect(url_for('index'))
        else:
            return render_template('login.html', error='Invalid credentials')
    return render_template('login.html')

# Logout route
@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('index'))

# Upload Video (protected route)
@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if 'username' not in session:
        return redirect(url_for('login'))
    if request.method == 'POST':
        file = request.files['video']
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            save_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            # Prevent overwriting files
            counter = 1
            original_filename = filename
            while os.path.exists(save_path):
                filename = f"{os.path.splitext(original_filename)[0]}_{counter}{os.path.splitext(original_filename)[1]}"
                save_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
                counter += 1
            file.save(save_path)
            return redirect(url_for('index'))
        else:
            return render_template('upload.html', error='Invalid file type', logged_in='username' in session)
    return render_template('upload.html', logged_in='username' in session)

# Serve video files
@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

# Run the app
if __name__ == '__main__':
    # Ensure uploads folder exists
    if not os.path.exists(UPLOAD_FOLDER):
        os.makedirs(UPLOAD_FOLDER)
    app.run(debug=True)
```

---

### 3. HTML Templates

#### `templates/index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Home - Video Site</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>
<body>
    <header>
        <h1>Welcome to the Video Site</h1>
        <nav>
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('about') }}">About</a>
            <a href="{{ url_for('contact') }}">Contact Us</a>
            {% if logged_in %}
                <a href="{{ url_for('upload') }}">Upload Video</a>
                <a href="{{ url_for('logout') }}">Logout</a>
            {% else %}
                <a href="{{ url_for('login') }}">Login</a>
            {% endif %}
        </nav>
    </header>
    <main>
        <h2>Videos</h2>
        {% if videos %}
            {% for video in videos %}
                <div class="video-container">
                    <video width="320" height="240" controls>
                        <source src="{{ url_for('uploaded_file', filename=video) }}">
                        Your browser does not support the video tag.
                    </video>
                    <p>{{ video }}</p>
                </div>
            {% endfor %}
        {% else %}
            <p>No videos uploaded yet.</p>
        {% endif %}
    </main>
    <footer>
        <p>&copy; 2024 Video Site</p>
    </footer>
</body>
</html>
```

#### `templates/about.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>About - Video Site</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>
<body>
    <header>
        <h1>About Us</h1>
        <nav>
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('about') }}">About</a>
            <a href="{{ url_for('contact') }}">Contact Us</a>
            {% if logged_in %}
                <a href="{{ url_for('upload') }}">Upload Video</a>
                <a href="{{ url_for('logout') }}">Logout</a>
            {% else %}
                <a href="{{ url_for('login') }}">Login</a>
            {% endif %}
        </nav>
    </header>
    <section>
        <p>This is a demo website where users can watch and upload videos.</p>
    </section>
</body>
</html>
```

#### `templates/contact.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Contact Us - Video Site</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>
<body>
    <header>
        <h1>Contact Us</h1>
        <nav>
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('about') }}">About</a>
            <a href="{{ url_for('contact') }}">Contact Us</a>
            {% if logged_in %}
                <a href="{{ url_for('upload') }}">Upload Video</a>
                <a href="{{ url_for('logout') }}">Logout</a>
            {% else %}
                <a href="{{ url_for('login') }}">Login</a>
            {% endif %}
        </nav>
    </header>
    <section>
        <p>For inquiries, email us at contact@videosite.com.</p>
    </section>
</body>
</html>
```

#### `templates/login.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Login - Video Site</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>
<body>
    <h1>Login</h1>
    {% if error %}
        <p class="error">{{ error }}</p>
    {% endif %}
    <form method="post" action="{{ url_for('login') }}">
        <label for="username">Username:</label><br>
        <input type="text" id="username" name="username" required /><br>
        <label for="password">Password:</label><br>
        <input type="password" id="password" name="password" required /><br><br>
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

#### `templates/upload.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Upload Video - Video Site</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>
<body>
    <h1>Upload a Video</h1>
    {% if error %}
        <p class="error">{{ error }}</p>
    {% endif %}
    <form method="post" enctype="multipart/form-data">
        <label for="video">Select video file:</label><br>
        <input type="file" id="video" name="video" accept="video/*" required /><br><br>
        <button type="submit">Upload</button>
    </form>
</body>
</html>
```

---

### 4. `static/style.css` (Basic Styling)
```css
body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}
header {
    background-color: #333;
    padding: 10px 0;
    color: #fff;
    text-align: center;
}
nav a {
    color: #fff;
    margin: 0 15px;
    text-decoration: none;
}
nav a:hover {
    text-decoration: underline;
}
main {
    padding: 20px;
}
.video-container {
    display: inline-block;
    margin: 10px;
    padding: 10px;
    background-color: #fff;
    border-radius: 4px;
}
footer {
    text-align: center;
    padding: 10px 0;
    background-color: #333;
    color: #fff;
}
.error {
    color: red;
}
```

---

## How to Run

1. Install Flask:
```bash
pip install flask
```

2. Save all files in the directory structure described.

3. Run the Flask app:
```bash
python app.py
```

4. Access the site via `http://127.0.0.1:5000/` in your browser.

---

## Summary
- Users can see videos on the homepage.
- Registered users can log in.
- Logged-in users can upload videos.
- Videos are stored locally and displayed dynamically.
- Navigation links allow moving between pages.
- Basic authentication is implemented for demonstration.

---

## Notes for Production
- For real deployment, replace the dummy user system with a database.
- Implement security measures, e.g., password hashing.
- Store videos in a dedicated storage system or cloud.
- Add pagination for videos if needed.
- Improve UI/UX for better responsiveness.

Let me know if you'd like a version with database integration or additional features!
