FROM python:3.11-slim

WORKDIR /app

# Copy dependencies
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy NLP service source
COPY . .

# Download required NLTK data
RUN python download_nltk_data.py

EXPOSE 5000

# Production Flask server
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:5000", "--workers", "2", "--timeout", "60"]