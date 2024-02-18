import datetime as dt
import json

import requests
from flask import Flask, jsonify, request

# create your API token, and set it up in Postman collection as part of the Body section
API_TOKEN = "ThisIsMyAPIToken"
# API key
RSA_KEY = "MNDRQW67J5SKPVZZL349M2B5F"

app = Flask(__name__)


class InvalidUsage(Exception):
    status_code = 400

    def __init__(self, message, status_code=None, payload=None):
        Exception.__init__(self)
        self.message = message
        if status_code is not None:
            self.status_code = status_code
        self.payload = payload

    def to_dict(self):
        rv = dict(self.payload or ())
        rv["message"] = self.message
        return rv


def weather_forecast(location, date):
    url = f"https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/{location}/{date}T13:00:00?key={RSA_KEY}&include=current"
    
    headers = {"X-Api-Key": RSA_KEY}

    response = requests.get(url)

    if response.status_code == requests.codes.ok:
        return json.loads(response.text)
    else:
        raise InvalidUsage(response.text, status_code=response.status_code)

@app.errorhandler(InvalidUsage)
def handle_invalid_usage(error):
    response = jsonify(error.to_dict())
    response.status_code = error.status_code
    return response


@app.route("/")
def home_page():
    return "<p><h2>KMA H1: Weather Saas.</h2></p>"


@app.route("/content/api/v1/integration/generate", methods=["POST"])
def weather_endpoint():
    start_dt = dt.datetime.now()
    json_data = request.get_json()
    location = json_data.get("location")
    date = json_data.get("date")

    if json_data.get("token") is None:
        raise InvalidUsage("token is required", status_code=400)

    token = json_data.get("token")

    if token != API_TOKEN:
        raise InvalidUsage("wrong API token", status_code=403)

    exclude = ""
    if json_data.get("exclude"):
        exclude = json_data.get("exclude")

    weather_data = weather_forecast(location, date)

    end_dt = dt.datetime.now()

    result = {
        "requester_name": json_data.get("requester_name"),
        "timestamp": end_dt.isoformat(),
        "location": location,
        "date": date,
        "weather": weather_data,
    }

    return result
