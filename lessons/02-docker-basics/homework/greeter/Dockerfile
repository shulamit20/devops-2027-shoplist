FROM python:3.11-slim

WORKDIR /app

COPY netfree.crt /usr/local/share/ca-certificates/netfree.crt
RUN update-ca-certificates

COPY requirements.txt .
ENV PIP_CERT=/etc/ssl/certs/ca-certificates.crt
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8000

CMD ["python", "app.py"]