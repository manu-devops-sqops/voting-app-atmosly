# Define a base stage that uses the official python runtime base image
FROM python:3.11-slim AS base

# Add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY vote/requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Define a stage specifically for development, where it'll watch for
# filesystem changes
FROM base AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
CMD ["python", "app.py"]

# Define the final stage that will bundle the application for production
FROM base AS final

# Copy our code from the current folder to the working directory inside the container
COPY /vote ./
COPY /vote/.env .

# Make port 80 available for links and/or publish
EXPOSE 80

# Load environment variables from .env file
ENV ENV_FILE_PATH .env
ENV $(cat $ENV_FILE_PATH | xargs)

# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]

# Set environment variables for the vote service
ENV REDIS_HOST=${REDIS_HOST}
ENV REDIS_PORT=${REDIS_PORT}
ENV REDIS_DB=${REDIS_DB}
ENV OPTION_A=${OPTION_A}
ENV OPTION_B=${OPTION_B}

# Specify the dependencies for the vote service (using HEALTHCHECK)
HEALTHCHECK --interval=15s --timeout=5s --retries=3 --start-period=10s CMD curl -f http://localhost || exit 1


