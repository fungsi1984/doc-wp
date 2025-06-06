# WordPress + Nginx + MySQL Docker Compose Setup

This project provides a complete local development environment for WordPress using Docker Compose, Nginx, and MySQL.

## Project Structure

```
docker-compose.yml
nginx/
  default.conf
```

## Services Overview

- **db**: MySQL 5.7 database for WordPress data storage.
- **wordpress**: WordPress running on PHP-FPM (php8.2-fpm) for compatibility with Nginx.
- **nginx**: Nginx web server serving WordPress and handling PHP requests via FPM.

## Step-by-Step Docker Compose Explanation

### 1. MySQL Service (`db`)
- Uses the `mysql:5.7` image.
- Sets up a database named `wordpress` with user `wordpress` and password `wordpress`.
- Data is persisted in a Docker named volume `db_data` (stored locally, not in the cloud).
- Exposes the database only to other containers via the `wpnet` network.

### 2. WordPress Service (`wordpress`)
- Uses the `wordpress:php8.2-fpm` image for PHP-FPM support (required for Nginx).
- Connects to the MySQL database using environment variables.
- WordPress files are stored in the `wp_data` Docker volume for persistence.
- Runs on the same custom Docker network `wpnet`.

### 3. Nginx Service (`nginx`)
- Uses the latest `nginx` image.
- Serves content from the same `wp_data` volume as WordPress, but mounted as read-only (`:ro`).
- Uses a custom Nginx config (`nginx/default.conf`) to handle PHP requests via FPM.
- Forwards port 8080 on your host to port 80 in the container, so you can access WordPress at [http://localhost:8080](http://localhost:8080).

### 4. Volumes
- `db_data`: Persists MySQL data locally (see `/var/lib/docker/volumes/db_data/_data`).
- `wp_data`: Persists WordPress files locally and shares them between WordPress and Nginx containers.

### 5. Networks
- `wpnet`: Custom bridge network for secure internal communication between containers.

## How to Use

1. **Start the stack:**
   ```zsh
   docker-compose up -d
   ```
2. **Stop the stack:**
   ```zsh
   docker-compose down
   ```
3. **Access WordPress:**
   Open [http://localhost:8080](http://localhost:8080) in your browser.

## Accessing Adminer (Database Management)

Adminer is included for easy MySQL database management.

- Open your browser and go to: [http://localhost:8081](http://localhost:8081)
- Use the following credentials to log in:

  - **System:** MySQL
  - **Server:** db
  - **Username:** wordpress
  - **Password:** wordpress
  - **Database:** wordpress

> Note: Use `db` as the server name because Adminer connects to the MySQL container via the Docker network.

## Using Adminer for SQL Queries and Database Management

Adminer provides a simple web interface for managing your MySQL database. Here’s how to use it for common tasks:

### 1. Logging In
- Open Adminer in your browser: [http://localhost:8081](http://localhost:8081)
- Enter your database credentials:
  - **System:** MySQL
  - **Server:** db
  - **Username:** wordpress
  - **Password:** wordpress
  - **Database:** wordpress

### 2. Running SQL Queries
- After logging in, click on **SQL command** in the left menu.
- Enter your SQL query in the text box and click **Execute**.

### 3. Common SQL Queries for WordPress
- **View all posts:**
  ```sql
  SELECT * FROM wp_posts LIMIT 10;
  ```
- **Find all users:**
  ```sql
  SELECT * FROM wp_users;
  ```
- **Update a user’s email:**
  ```sql
  UPDATE wp_users SET user_email='new@email.com' WHERE ID=1;
  ```
- **Delete a comment:**
  ```sql
  DELETE FROM wp_comments WHERE comment_ID=123;
  ```

### 4. Other Adminer Features
- **Browse tables:** Click a table name to view, edit, or delete rows.
- **Import/Export:** Use the Import or Export links to move data in/out.
- **Create/Drop tables:** Use Create table or Drop to manage structure.

### 5. Learning SQL Basics
- Start with `SELECT`, `INSERT`, `UPDATE`, `DELETE` statements.
- Practice filtering with `WHERE`, sorting with `ORDER BY`, and joining tables with `JOIN`.

> For more SQL examples or a quick tutorial, just ask!

## Exporting and Importing WordPress Posts (Database Content)

WordPress posts, pages, users, and settings are stored in the MySQL database, not as files. To back up or move your content, you need to export/import the database. You can do this easily with Adminer, which is included in this setup.

### Exporting the Database with Adminer
1. Open Adminer at [http://localhost:8081](http://localhost:8081).
2. Log in with:
   - **System:** MySQL
   - **Server:** db
   - **Username:** wordpress
   - **Password:** wordpress
   - **Database:** wordpress
3. Click the **Export** link in the left menu.
4. Choose **SQL** as the output format.
5. Click **Export** to download a .sql file containing your entire database (including posts, pages, users, etc.).

### Importing the Database with Adminer
1. Open Adminer at [http://localhost:8081](http://localhost:8081).
2. Log in with the same credentials as above.
3. Click the **Import** link in the left menu.
4. Choose your .sql file (from a previous export or another WordPress site).
5. Click **Execute** to import the database content.

> **Note:** Importing a database will overwrite existing data. Always back up your current database before importing.

### Export/Import via WordPress Admin (for Posts Only)
Alternatively, you can use the built-in WordPress Tools > Export/Import feature to move only posts, pages, or media (not full database):
1. Go to **Tools > Export** in the WordPress admin dashboard to download an XML file of your posts/pages.
2. Go to **Tools > Import** on another site to upload the XML file.

## Exporting and Importing the WordPress Database with mysqldump

You can also use `mysqldump` to export and import your WordPress database directly from the command line. This is useful for backups, migrations, or advanced workflows.

### Exporting the Database
Run this command from your project directory to export the database to a file named `wordpress.sql`:

```zsh
docker exec mysql mysqldump -uwordpress -pwordpress wordpress > wordpress.sql
```
- `mysql` is the name of your database container.
- `-uwordpress -pwordpress` are the username and password (change if you use different credentials).
- `wordpress` at the end is the database name.

### Importing the Database
To import a previously exported `wordpress.sql` file:

```zsh
docker exec -i mysql mysql -uwordpress -pwordpress wordpress < wordpress.sql
```

> **Note:** This will overwrite the current database content. Always back up your data before importing.

### Alternative: Use Adminer
You can also use Adminer at [http://localhost:8081](http://localhost:8081) for database export/import via a web interface (see above for details).

---

## Notes
- The `:ro` (read-only) flag on the Nginx volume ensures Nginx cannot modify WordPress files, improving security.
- The `version` attribute in `docker-compose.yml` is omitted as it is now obsolete.
- All data is stored locally unless you configure Docker to use remote/cloud volumes.

## Troubleshooting
- If you see a `502 Bad Gateway` error, ensure the WordPress service uses the FPM image and that all containers are running.
- To view logs for a service:
  ```zsh
  docker-compose logs <service>
  ```
  Replace `<service>` with `db`, `wordpress`, or `nginx`.

## Production Best Practices & SSL (HTTPS) Setup

### Why Use OpenSSL?
- **OpenSSL** is used to generate SSL/TLS certificates for HTTPS. For local development, you can use self-signed certificates. For production, use trusted certificates (e.g., from Let's Encrypt).

### Enabling HTTPS with Nginx and Docker Compose
1. **Generate Self-Signed Certificates (for local testing):**
   ```zsh
   mkdir -p nginx/certs
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout nginx/certs/selfsigned.key \
     -out nginx/certs/selfsigned.crt \
     -subj "/CN=localhost"
   ```
2. **Update Nginx Config:**
   Add an `ssl` server block in `nginx/default.conf`:
   ```nginx
   server {
       listen 443 ssl;
       server_name localhost;
       ssl_certificate /etc/nginx/certs/selfsigned.crt;
       ssl_certificate_key /etc/nginx/certs/selfsigned.key;
       # ...rest of config (same as HTTP block)...
   }
   ```
   And mount the certs in your `docker-compose.yml`:
   ```yaml
   volumes:
     - ./nginx/certs:/etc/nginx/certs:ro
   ```
3. **For Production:**
   - Use Let's Encrypt for free, trusted SSL certificates (see [Certbot](https://certbot.eff.org/)).
   - Automate certificate renewal.

### Additional Production Best Practices
- **Do not expose MySQL to the public.**
- **Use strong, unique passwords and secrets.**
- **Back up your database and WordPress files regularly.**
- **Keep Docker images and plugins up to date.**
- **Limit file permissions and use `:ro` where possible.**
- **Do not hardcode secrets in your compose file; use environment files or Docker secrets.**
- **Run containers as non-root users when possible.**
- **Disable directory listing in Nginx.**

### Generating a Self-Signed SSL Certificate with OpenSSL

To generate a self-signed SSL certificate and private key for local HTTPS development, run the following command in your project root:

```zsh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/certs/selfsigned.key \
  -out nginx/certs/selfsigned.crt \
  -subj "/CN=localhost"
```

- This will create `nginx/certs/selfsigned.crt` and `nginx/certs/selfsigned.key`.
- These files are mounted into the Nginx container for HTTPS support.
- **Do not use self-signed certificates in production.**

## Example Self-Signed SSL Certificate and Key

Below are example contents for self-signed certificate and key files used for local HTTPS with Nginx. **Do not use these in production.**

### nginx/certs/selfsigned.crt
```
-----BEGIN CERTIFICATE-----
MIIDCTCCAfGgAwIBAgIUT+7QzPlJeehjdQGF9wKPqdXjp3owDQYJKoZIhvcNAQEL
BQAwFDESMBAGA1UEAwwJbG9jYWxob3N0MB4XDTI1MDUyMzE0NDIyNVoXDTI2MDUy
MzE0NDIyNVowFDESMBAGA1UEAwwJbG9jYWxob3N0MIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAy6ZXDXHxSiSgZhDe8XDDwHRNR9xiU7A5aFsDxyV/t2ZV
kdv2YGGVwXfgPLdjY6hCUKSpxuUZY3hNRNMT7s7LKo5ctm64Skf3tlVYU0fF4Oys
zHccvTKEYRKcm8T8VcUzyYIWAtteik2g4STIaTbkgkB5OwiAUvnPLBQCv+gW0emU
pKwFcKRmOhRNtcy7r1NefGx2oitT9vb2ncI+WjrP2xgEvyB7GKHaE3pdqkm1OG/Q
Wcst01UOOxAftTXF55ogVQ5j2QoLlg4e6zOeJBhXkdHqkoQv3P6s+zdXM8Bpx5Ar
Sqz1GcBBRiAtkoeSJrE6QSUpHm6tHsgKpipOHSXvbQIDAQABo1MwUTAdBgNVHQ4E
FgQUap1BeOZw49+kamLFRNP5wNOiW3owHwYDVR0jBBgwFoAUap1BeOZw49+kamLF
RNP5wNOiW3owDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEAn61C
+Kw/+ORxkRNlGgZCuqSdfeDiZY0fDxtxfE2LLHJQJ31JavJkcmzr6n30e3XXYZQk
gZ3kCdbbbcXAQjniR2n9FHSFAeLy5WlpZQD8FQ1spaRjTZ3KgAtO9X2QlmF8cgEP
/Aw4uEXSnJol5IoRiusfOFSBVmxkCV9SjbRPCThqUEH4dDThdJCYbSGf3DmMUU3/
8qxhdS412nEv01o4v5yAgHLWA0LgkfsOXaR03a666M4mG/Hr04DEWTpmyF+4g68i
/mvqs9l9T/H7S20FCc+51Cc3yD1QWGeXVFfSaz2B2x/AFVUNr7TSOAja0tN+BWlA
M3Lld+JvJQE3E7YYAg==
-----END CERTIFICATE-----
```

## Production Deployment Recommendations

For a secure and reliable production WordPress deployment, follow these best practices:

### 1. Use Trusted SSL Certificates
- Obtain SSL certificates from a trusted Certificate Authority (e.g., Let's Encrypt).
- Automate certificate renewal with tools like Certbot.
- Update your Nginx config to use the trusted certificate and key.

### 2. Secure Secrets and Environment Variables
- Do not hardcode passwords or secrets in `docker-compose.yml`.
- Use Docker secrets, environment files (`.env`), or a secrets manager for sensitive data.
- Restrict access to these files and rotate secrets regularly.

### 3. Database Security
- Never expose MySQL to the public internet. Only allow access from trusted containers or hosts.
- Use strong, unique passwords for all database users.
- Consider restricting MySQL user privileges to only what's necessary.

### 4. File Permissions & User Security
- Run containers as non-root users where possible (see Docker image documentation for options).
- Set strict file and directory permissions, especially for WordPress files and uploads.
- Use the `:ro` (read-only) flag for Nginx and other services that do not need write access.

### 5. Keep Everything Updated
- Regularly update Docker images, WordPress core, plugins, and themes to patch security vulnerabilities.
- Subscribe to security advisories for your stack.

### 6. Backups
- Automate regular backups of your database and WordPress files.
- Store backups securely, offsite if possible, and test restore procedures.

### 7. Nginx Hardening
- Disable directory listing.
- Limit request size and rate to prevent abuse (e.g., `client_max_body_size`, rate limiting modules).
- Set security headers (e.g., `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`).
- Consider using a Web Application Firewall (WAF) or a reverse proxy with WAF capabilities.

### 8. Monitoring & Logging
- Enable and monitor logs for Nginx, PHP, and MySQL.
- Use monitoring tools to track uptime, performance, and security events.

### 9. Disable Unused Services
- Remove Adminer or phpMyAdmin from production deployments, or restrict access (e.g., with authentication or IP whitelisting).

### 10. Additional WordPress Security
- Change the default database table prefix.
- Use reputable security plugins (e.g., Wordfence).
- Disable file editing from the WordPress admin panel (`define('DISALLOW_FILE_EDIT', true);` in `wp-config.php`).
- Limit login attempts and use strong admin passwords.
- Consider two-factor authentication for admin users.

---

**Summary:**
- Use trusted SSL certificates and automate renewal.
- Secure all secrets and environment variables.
- Harden your database, Nginx, and WordPress configuration.
- Keep everything updated and automate backups.
- Remove or restrict access to database management tools in production.

For a sample production-ready `docker-compose.yml` or hardened Nginx config, just ask!

### Using Let's Encrypt SSL Certificates in Production

To use trusted SSL certificates from Let's Encrypt in your Docker Compose setup:

1. **Obtain certificates with Certbot on your server:**
   ```zsh
   sudo apt update
   sudo apt install certbot
   sudo certbot certonly --standalone -d yourdomain.com
   ```
   This will generate certificates in `/etc/letsencrypt/live/yourdomain.com/`.

2. **Update your `docker-compose.yml`:**
   Uncomment and edit the following lines in the `nginx` service:
   ```yaml
   #   - /etc/letsencrypt/live/yourdomain.com/fullchain.pem:/etc/nginx/certs/fullchain.pem:ro
   #   - /etc/letsencrypt/live/yourdomain.com/privkey.pem:/etc/nginx/certs/privkey.pem:ro
   ```
   Replace `yourdomain.com` with your actual domain name.

3. **Update your Nginx config (`nginx/default.conf`):**
   Change the SSL certificate paths:
   ```nginx
   ssl_certificate /etc/nginx/certs/fullchain.pem;
   ssl_certificate_key /etc/nginx/certs/privkey.pem;
   ```

4. **Restart your containers:**
   ```zsh
   docker compose down
   docker compose up -d
   ```

5. **Automate certificate renewal:**
   Set up a cron job to renew certificates and restart Nginx:
   ```zsh
   sudo crontab -e
   ```
   Add:
   ```
   0 3 * * * certbot renew --quiet && docker compose restart nginx
   ```

---

This will ensure your production site uses trusted SSL certificates and stays secure.

## Viewing PHP Configuration with phpinfo()

To check your PHP configuration inside the Docker container, you can use the phpinfo() function:

1. Create a file named `phpinfo.php` with the following contents:
   ```php
   <?php
   phpinfo();
   ```
2. Place this file in the root WordPress directory (the same location as `wp-config.php`).
   - If you place it in `wp-content`, it may not be accessible due to Nginx configuration.
   ```
   example 
   docker cp /home/desktop/Documents/doc-wp/wp-content/phpinfo.php wordpress:/var/www/html/phpinfo.php
   ```
3. Access it in your browser at: [http://localhost:8080/phpinfo.php](http://localhost:8080/phpinfo.php)
4. **Important:** Delete this file after use for security reasons.

> If you see "file not found" when accessing `phpinfo.php` in `wp-content`, move it to the root WordPress directory as described above.

---
