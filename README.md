# ATS-API-PYTHON
## Contents
### 1.Objective
### 2.Project Structure
### 3.Project overview
### 4.CODE
### 5.Problem Statement
## OBJECTIVE
#### I GOT INTERESTED, TO KNOW HOW THESE COMPANIES ARE USING APPLICATION TRACKING SYSTEM(ATS) AND ITS WORKING. SO I DEVELOPED REAL TIME PROJECT BY USING PYTHON WITH FLASK, THAT READS THE RESUME AND TO FIND THE PARTICULAR KEYWORDS THAT COMPANY WANTS.

## PROJECT STRUCTURE
```
ATS-API/
├── app.py                   # Main Flask application file
├── templates/
│   └── index.html           # HTML file for the user interface
├── uploads/                 # Directory to temporarily store uploaded resumes
├── keywords/                # Folder containing job role-specific keyword files
│   ├── data_analyst.txt
│   ├── software_developer.txt
│   ├── ui_ux_designer.txt
│   ├── web_developer.txt
│   ├── cloud_engineer.txt
│   └── network_engineer.txt
```
## PROJECT OVERVIEW
#### -The user opens their browser and navigates to 127.0.0.1:8080.
#### -The html page is displayed, providing a simple and interactive user interface.
#### -The user uploads a resume using the file input on the webpage.
#### -They select the job role (e.g., Data Analyst, Software Developer) from a dropdown menu.
#### -After making the selection, they click the "Submit" button.
#### -The uploaded resume is temporarily saved in the uploads/ folder for further processing.
#### -The selected job role determines which keyword file (e.g., data_analyst.txt) is loaded from the keywords/ directory.
#### -The backend extracts text from the uploaded resume (e.g., work experience, education, skills).This parsing is handled by the logic in app.py.
#### -The parsed resume data is compared against the keywords for the selected job role.
#### -Then,user sees the result like "keywords matched for selected job role:_,_,_". If keywords not matched user sees the message like "No keywords matched".
#### -After processing, the uploaded file is deleted from the uploads/ folder.

## CODE

``` HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ATS Application</title>
</head>
<body>
    <h1>Resume Checker</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <!-- Job Role Dropdown -->
        <label for="job_role">Select Job Role:</label>
        <select name="job_role" id="job_role" required>
            <option value="Data Analyst">Data Analyst</option>
            <option value="Software Developer">Software Developer</option>
            <option value="UI/UX Designer">UI/UX Designer</option>
            <option value="Web Developer">Web Developer</option>
            <option value="Cloud Engineer">Cloud Engineer</option>
            <option value="Network Engineer">Network Engineer</option>
        </select>
        <br><br>
        
        <!-- File Upload -->
        <label for="resume">Upload Resume:</label>
        <input type="file" name="resume" id="resume" accept=".pdf" required>
        <br><br>
        
        <!-- Submit Button -->
        <button type="submit">Check</button>
    </form>
</body>
</html>
```
---------------------------------------------------------------

``` PYTHON
from flask import Flask, render_template, request
from waitress import serve
from PyPDF2 import PdfReader
import os

app = Flask(__name__)

KEYWORDS_FOLDER = 'keywords'  # Directory containing job role keywords

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/upload', methods=['POST'])
def upload():
    # Get job role and file from the form
    job_role = request.form['job_role']
    uploaded_file = request.files['resume']
    
    # Check if a file is uploaded
    if not uploaded_file:
        return "No file uploaded!"
    
    # Check if the uploaded file is a PDF
    if not uploaded_file.filename.endswith('.pdf'):
        return "Please upload a valid PDF file."

    # Read the uploaded file in memory
    try:
        reader = PdfReader(uploaded_file)
        pdf_text = " ".join([page.extract_text() for page in reader.pages])
    except Exception as e:
        return f"Error reading the file: {e}"

    # Construct the path for the keywords file
    # Ensure .txt is not appended if the file already has it
    keywords_filename = f"{job_role.lower().replace(' ', '_')}"
    if not keywords_filename.endswith('.txt'):
        keywords_filename += '.txt'
    
    keywords_path = os.path.join(KEYWORDS_FOLDER, keywords_filename)

    try:
        with open(keywords_path, 'r') as f:
            keywords = [line.strip().lower() for line in f.readlines()]
    except FileNotFoundError:
        return f"No keywords file found for {job_role}!"

    # Match keywords in the resume
    matches = [keyword for keyword in keywords if keyword in pdf_text.lower()]
    if matches:
        return f"Matched keywords for {job_role}: {', '.join(matches)}"
    else:
        return f"No keywords matched for {job_role} role."

if __name__ == '__main__':
    print("Starting Flask app with Waitress...")
    serve(app, host='0.0.0.0', port=8080)
```
## 
