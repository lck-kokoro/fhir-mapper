# FHIR Application Deployment Guide

## Prerequisites
- Ensure you have Java installed. You can verify your Java version using:
  ```sh
  java -version
  ```
- Install Gradle if not already installed.
- Ensure you have `nginx` installed and running.

## Build and Run the Application

1. **Build the Application**
   ```sh
   gradle build --no-daemon -x test
   ```
   This will build the project while skipping the test execution.

2. **Locate the JAR File**
   ```sh
   ls build/libs/
   ```
   The output will list the generated JAR file, usually named with a `-SNAPSHOT` suffix.

3. **Copy the JAR File**
   ```sh
   cp build/libs/*-SNAPSHOT.jar ./fhir.jar
   ```
   This command renames and moves the built JAR file to the current directory as `fhir.jar`.

4. **Run the Application**
   ```sh
   java -jar fhir.jar
   ```
   Alternatively, you can use `pm2` for process management:
   ```sh
   pm2 start "java -jar fhir.jar" --name "fhir-app"
   ```
   This will ensure the application keeps running even after terminal closure.

## Configure Nginx as a Reverse Proxy

1. **Edit Nginx Configuration**
   Add the following configuration to your Nginx settings (e.g., in `/etc/nginx/sites-available/default` or `/etc/nginx/nginx.conf`):
   ```nginx
   server {
       listen 80;
       server_name example.com www.example.com;
   
       location / {
           proxy_pass http://localhost:8085;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

2. **Reload Nginx**
   Apply the new configuration by running:
   ```sh
   sudo systemctl reload nginx
   ```

## Notes
- Replace `example.com` with your actual domain name.
- Ensure your application runs on port `8085` or update the Nginx config accordingly.
- If using `pm2`, you may need to run `pm2 save` to persist the process across reboots.

## Troubleshooting
- Check if Java is installed properly using `java -version`.
- Ensure the JAR file is built successfully and located in `build/libs/`.
- If the application does not start, check logs using:
  ```sh
  pm2 logs fhir-app
  ```
- Restart Nginx if necessary:
  ```sh
  sudo systemctl restart nginx
  ```


