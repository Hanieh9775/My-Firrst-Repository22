import os
import openai
from flask import Flask, request, render_template_string, redirect, url_for
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.config["UPLOAD_FOLDER"] = "uploads"
os.makedirs(app.config["UPLOAD_FOLDER"], exist_ok=True)
openai.api_key = os.getenv("OPENAI_API_KEY")  # Set your OpenAI API key

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AI Image Caption Generator</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: #f4f6f8; padding: 50px; }
        .container { max-width: 700px; background: white; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); padding: 30px; }
        img { max-width: 100%; border-radius: 10px; margin-top: 20px; }
        .caption { background: #f8f9fa; border-radius: 10px; padding: 15px; margin-top: 15px; white-space: pre-wrap; }
    </style>
</head>
<body>
    <div class="container text-center">
        <h2 class="mb-4">🖼️ AI Image Caption Generator</h2>
        <form method="post" enctype="multipart/form-data">
            <input type="file" name="image" accept="image/*" class="form-control mb-3" required>
            <button class="btn btn-primary w-100">Generate Caption</button>
        </form>
        {% if image_url %}
            <img src="{{ image_url }}" alt="Uploaded Image">
        {% endif %}
        {% if caption %}
            <div class="caption">{{ caption }}</div>
        {% endif %}
    </div>
</body>
</html>
"""

def generate_caption(image_path):
    prompt = f"Describe this image in one short and creative sentence: {image_path}"
    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7,
        max_tokens=80
    )
    return response["choices"][0]["message"]["content"].strip()

@app.route("/", methods=["GET", "POST"])
def index():
    image_url, caption = None, None
    if request.method == "POST":
        file = request.files["image"]
        if file:
            filename = secure_filename(file.filename)
            path = os.path.join(app.config["UPLOAD_FOLDER"], filename)
            file.save(path)
            image_url = url_for("static", filename=filename)
            caption = generate_caption(filename)
    return render_template_string(HTML_TEMPLATE, image_url=image_url, caption=caption)

if __name__ == "__main__":
    app.run(debug=True)
