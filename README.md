from flask import Flask, request, jsonify
from config import Config
from models import mongo, save_event
from datetime import datetime

app = Flask(__name__)
app.config["MONGO_URI"] = Config.MONGO_URI

# Initialize PyMongo
mongo.init_app(app)

@app.route("/webhook", methods=["POST"])
def webhook():
    try:
        event_data = request.get_json()

        if not event_data:
            return jsonify({"error": "Invalid JSON"}), 400

        # Add receivedAt timestamp
        event_data["receivedAt"] = datetime.utcnow()

        save_event(event_data)
        print("Event saved:", event_data)

        return jsonify({"status": "success"}), 200

    except Exception as e:
        print("Error:", str(e))
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
