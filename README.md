# Locfinder-
from flask import Flask, render_template, request
import socket
import requests
import ipaddress

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def home():
    ip = None
    location = None
    error = None
    domain = ""

    if request.method == "POST":
        domain = request.form.get("domain", "").strip()

        try:
            # Accept a domain, not a full URL
            domain = domain.replace("https://", "").replace("http://", "")
            domain = domain.split("/")[0].split(":")[0].strip()

            if not domain or any(c.isspace() for c in domain):
                raise ValueError("Please enter a valid domain name.")

            # Find the IP address
            ip = socket.gethostbyname(domain)

            # Find approximate server location
            ip_obj = ipaddress.ip_address(ip)

            if not ip_obj.is_global:
                raise ValueError(
                    "This domain resolved to a private or reserved IP."
                )

            response = requests.get(
                f"https://ipinfo.io/{ip}/json",
                timeout=10
            )
            response.raise_for_status()
            data = response.json()

            location = {
                "country": data.get("country", "Unknown"),
                "city": data.get("city", "Unknown"),
                "region": data.get("region", "Unknown"),
                "coordinates": data.get("loc", "Unknown"),
                "organization": data.get("org", "Unknown")
            }

        except Exception:
            ip = None
            location = None
            error = (
                "Unable to find the location. "
                "Check the domain name and try again."
            )

    return render_template(
        "index.html",
        ip=ip,
        location=location,
        error=error,
        domain=domain
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
