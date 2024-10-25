FROM python:3.12.0b3-alpine3.18

# Set the working directory
WORKDIR /application

# Copy everything from the current context (the Docker_Pipeline directory) to /application
COPY . .

# List the files in the working directory for debugging
RUN ls -al

# Install the requirements
RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8080

CMD ["python", "app.py"]
