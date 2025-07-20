from flask import Flask, request, jsonify
from flask_cors import CORS
from gtts import gTTS
from deep_translator import GoogleTranslator
import os
import uuid
import re

app = Flask(__name__)
CORS(app)

# Ensure 'static' folder exists
if not os.path.exists("static"):
    os.makedirs("static")

# Mapping user-friendly accents to TLDs
TLD_MAP = {
    "us": "com",
    "uk": "co.uk",
    "india": "co.in",
    "australia": "com.au",
    "south_africa": "co.za"
}

@app.route("/speak", methods=["POST"])
def speak():
    try:
        # Get user input
        text = request.json.get('text', '').strip()
        accent = request.json.get("accent", "us").lower()

        if not text:
            return jsonify({"error": "No text provided"}), 400

        # Convert accent to TLD
        tld = TLD_MAP.get(accent, "com")

        # Add pause after each full stop for better flow in gTTS
        processed_text = re.sub(r'\.\s*', '. ... ', text)

        # Translate to Bengali
        translated = GoogleTranslator(source='auto', target='bn').translate(text)

        # Generate speech
        tts = gTTS(text=processed_text, lang='en', tld=tld)

        # Save audio file
        filename = f"audio_{uuid.uuid4().hex}.mp3"
        filepath = os.path.join("static", filename)
        tts.save(filepath)

        return jsonify({
            "translation": translated,
            "audio_url": f"/static/{filename}"
        })

    except Exception as e:
        print("Error:", e)
        return jsonify({"error": "Translation or TTS failed"}), 500

if __name__ == "__main__":
    app.run(debug=True)