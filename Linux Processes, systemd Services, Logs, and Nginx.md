# Linux Processes, systemd Services, Logs, and Nginx

## A Simple Beginner-Friendly Guide

This guide explains the following topics in a clear order:

1. What a process is
2. What a service is
3. What systemd is
4. What a systemd service file contains
5. How to create and manage a service
6. What stdin, stdout, and stderr mean
7. Where systemd service logs go by default
8. How to send logs to files
9. How to manage log file size
10. What Nginx is and where its logs go
11. The difference between Nginx logs and systemd logs
12. A complete application, systemd, and Nginx example
13. A troubleshooting checklist

---

# 1. What Is a Process?

A **process** is a program that is currently running on the computer.

For example, when you run:

```bash
python3 app.py
```

Linux starts a Python process for `app.py`.

You can see running processes with:

```bash
ps aux
```

You can search for a specific process:

```bash
ps aux | grep app.py
```

You can also view running processes with:

```bash
top
```

Every process has a unique number called a **PID**, which means **Process ID**.

Example:

```text
USER       PID  COMMAND
myapp     2451  python3 app.py
```

In this example:

- `myapp` is the user running the process.
- `2451` is the process ID.
- `python3 app.py` is the running command.

## Problem with starting a process manually

If you start an application manually:

```bash
python3 app.py
```

it may stop when:

- You close the terminal.
- You log out.
- The application crashes.
- The server restarts.

A service manager such as **systemd** solves these problems.

---

# 2. What Is a Service?

A **service** is usually a program that runs in the background for a long time and provides a function.

Examples include:

- Nginx web server
- SSH server
- MySQL database
- Docker
- A Python API
- A Node.js application
- A Java application

When systemd manages an application as a service, systemd can:

- Start the application.
- Stop the application.
- Restart the application.
- Start it automatically when Linux boots.
- Restart it when it fails.
- Run it as a specific Linux user.
- give it environment variables.
- Collect its standard output and errors.

For example, Nginx normally runs as a service called:

```text
nginx.service
```

Check it with:

```bash
sudo systemctl status nginx.service
```

## Process versus service

A **process** is a running program.

A **service** is a background function that is usually managed by a service manager such as systemd.

A service normally has one main process and may also have child processes.

---

# 3. What Is systemd?

**systemd** is the system and service manager used by many Linux distributions, including Ubuntu, Debian, Red Hat, Rocky Linux, AlmaLinux, and Fedora.

systemd starts very early during Linux boot. It normally runs as process number 1, also called **PID 1**.

Check PID 1 with:

```bash
ps -p 1 -o pid,comm,args
```

Example output:

```text
PID COMMAND ARGS
1   systemd /sbin/init
```

systemd is not your application. It is the manager that starts and monitors many services on the computer.

## What systemd does

systemd can:

1. Start services during boot.
2. Stop services during shutdown.
3. Start, stop, and restart applications.
4. Restart failed applications.
5. Manage dependencies between services.
6. Run applications as specific users and groups.
7. Collect application output through the system journal.
8. Track the processes that belong to each service.
9. Run scheduled tasks using systemd timers.

---

# 4. What Is a systemd Unit?

A **unit** is something managed by systemd.

Common unit types are:

```text
.service    A background service
.socket     A network or local socket
.timer      A scheduled task
.mount      A filesystem mount
.path       A watched file or directory
.target     A group of units
```

For an application, you normally create a file ending in:

```text
.service
```

Example:

```text
myapp.service
```

---

# 5. Where Are systemd Service Files Stored?

Custom service files are normally stored in:

```text
/etc/systemd/system/
```

Example:

```text
/etc/systemd/system/myapp.service
```

Package-installed service files are commonly stored in one of these locations:

```text
/usr/lib/systemd/system/
/lib/systemd/system/
```

For your own application, normally use:

```text
/etc/systemd/system/myapp.service
```

Do not directly edit a package service file in `/usr/lib/systemd/system/` or `/lib/systemd/system/`. A package update may replace your changes. Use a systemd override instead.

---

# 6. What Do We Write in a Service File?

A basic service file has three main sections:

```ini
[Unit]

[Service]

[Install]
```

Here is a complete example:

```ini
[Unit]
Description=My Python Web Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
Restart=on-failure
RestartSec=5
Environment="APP_ENV=production"
EnvironmentFile=-/etc/myapp/myapp.env
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

## 6.1 The `[Unit]` section

This section describes the service and its relationship with other units.

### `Description=`

```ini
Description=My Python Web Application
```

This is a readable name for the service. It appears in:

```bash
systemctl status myapp.service
```

### `After=`

```ini
After=network-online.target
```

This controls startup order. It tells systemd to start this service after the network is considered online.

`After=` controls order. It does not always cause the named unit to start.

### `Wants=`

```ini
Wants=network-online.target
```

This creates a weaker dependency on the named unit.

---

## 6.2 The `[Service]` section

This section tells systemd how to run and monitor the application.

### `Type=`

```ini
Type=simple
```

This means the command started by `ExecStart=` is the main service process.

This works well for many Python, Node.js, Java, and Go applications. The application should stay in the foreground.

### `User=` and `Group=`

```ini
User=myapp
Group=myapp
```

These lines tell systemd which Linux user and group should run the application.

Do not run an application as `root` unless it truly needs root permissions.

### `WorkingDirectory=`

```ini
WorkingDirectory=/opt/myapp
```

This tells systemd to use `/opt/myapp` as the application's current directory.

It is similar to running:

```bash
cd /opt/myapp
```

before starting the program.

### `ExecStart=`

```ini
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
```

This is the exact command systemd runs.

Use full paths. This is safer than:

```ini
ExecStart=python3 app.py
```

systemd does not normally run `ExecStart=` through a shell. Shell features such as `>`, `2>&1`, pipes, and command substitution do not work in the normal Bash way.

Use systemd logging settings instead of shell redirection.

### `Restart=`

```ini
Restart=on-failure
```

This tells systemd to restart the application when it exits with a failure.

Common values are:

```ini
Restart=no
Restart=always
Restart=on-failure
Restart=on-abnormal
```

For many applications, `on-failure` is a good choice.

### `RestartSec=`

```ini
RestartSec=5
```

systemd waits five seconds before restarting the failed application.

### `Environment=`

```ini
Environment="APP_ENV=production"
Environment="PORT=8000"
```

These lines define environment variables for the application.

### `EnvironmentFile=`

```ini
EnvironmentFile=-/etc/myapp/myapp.env
```

This loads environment variables from a separate file.

The leading `-` means the service can still start if the file does not exist.

Example `/etc/myapp/myapp.env`:

```text
APP_ENV=production
PORT=8000
DATABASE_HOST=127.0.0.1
```

Do not write `export` in a systemd environment file. Write:

```text
PORT=8000
```

not:

```text
export PORT=8000
```

Protect files that contain passwords or tokens:

```bash
sudo chown root:myapp /etc/myapp/myapp.env
sudo chmod 640 /etc/myapp/myapp.env
```

### `StandardOutput=` and `StandardError=`

```ini
StandardOutput=journal
StandardError=journal
```

These lines send normal output and error output to the systemd journal.

### `SyslogIdentifier=`

```ini
SyslogIdentifier=myapp
```

This gives the service log messages a recognizable name.

---

## 6.3 The `[Install]` section

```ini
[Install]
WantedBy=multi-user.target
```

This allows the service to be enabled for normal server startup.

---

# 7. How to Create and Start a systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/myapp.service
```

After saving it, check the unit file:

```bash
sudo systemd-analyze verify /etc/systemd/system/myapp.service
```

Tell systemd to read the new or changed service file:

```bash
sudo systemctl daemon-reload
```

Enable the service to start during boot:

```bash
sudo systemctl enable myapp.service
```

Start it now:

```bash
sudo systemctl start myapp.service
```

Enable it at boot and start it now with one command:

```bash
sudo systemctl enable --now myapp.service
```

Check its status:

```bash
sudo systemctl status myapp.service
```

Restart it:

```bash
sudo systemctl restart myapp.service
```

Stop it:

```bash
sudo systemctl stop myapp.service
```

Disable automatic startup:

```bash
sudo systemctl disable myapp.service
```

## Important difference

```bash
sudo systemctl start myapp.service
```

starts the service now.

```bash
sudo systemctl enable myapp.service
```

configures it to start during future boots.

```bash
sudo systemctl enable --now myapp.service
```

does both.

---

# 8. What Are stdin, stdout, and stderr?

Every Linux process normally has three standard streams:

```text
0 = stdin
1 = stdout
2 = stderr
```

## `stdin`: Standard Input

`stdin` is information going into the program.

For an interactive command, this can be what you type on the keyboard.

## `stdout`: Standard Output

`stdout` is the program's normal output.

Example:

```bash
echo "Application started"
```

The text is written to stdout.

Python example:

```python
print("Application started")
```

`print()` normally writes to stdout.

## `stderr`: Standard Error

`stderr` is normally used for errors, warnings, and diagnostic messages.

Example:

```bash
ls /directory-that-does-not-exist
```

The error is written to stderr.

Python example:

```python
import sys

print("Database connection failed", file=sys.stderr)
```

## Easy picture

```text
stdin  ---> application ---> stdout
                     |
                     +-----> stderr
```

## stdout and stderr do not decide success

stdout and stderr are output channels. They do not decide whether a command succeeded.

A program reports success or failure using an **exit status**:

```text
0       Usually means success
nonzero Usually means failure
```

Check the previous command's exit status with:

```bash
echo $?
```

A program can write an error-looking message to stdout, or a normal-looking message to stderr. The application chooses which stream to use.

---

# 9. Shell Redirection Examples

Save stdout to a file:

```bash
my-command > output.log
```

This is the same as:

```bash
my-command 1> output.log
```

Append stdout instead of replacing the file:

```bash
my-command >> output.log
```

Save stderr to a file:

```bash
my-command 2> error.log
```

Append stderr:

```bash
my-command 2>> error.log
```

Save stdout and stderr in different files:

```bash
my-command > output.log 2> error.log
```

Save both in one file:

```bash
my-command > combined.log 2>&1
```

Discard stdout:

```bash
my-command > /dev/null
```

Discard stderr:

```bash
my-command 2> /dev/null
```

Discard both:

```bash
my-command > /dev/null 2>&1
```

Do not discard errors in production unless you are sure they are not needed.

---

# 10. Where Do New systemd Service Logs Go by Default?

If you create a service like this:

```ini
[Service]
ExecStart=/usr/bin/python3 /opt/myapp/app.py
```

its stdout and stderr normally go to the **systemd journal**.

The effective behavior is normally similar to:

```ini
StandardOutput=journal
StandardError=journal
```

View the service logs:

```bash
sudo journalctl -u myapp.service
```

Follow them live:

```bash
sudo journalctl -u myapp.service -f
```

Show the newest 100 lines:

```bash
sudo journalctl -u myapp.service -n 100
```

Show logs from the current boot:

```bash
sudo journalctl -u myapp.service -b
```

Show logs since a specific time:

```bash
sudo journalctl -u myapp.service --since "2026-08-18 10:00:00"
```

For troubleshooting, use:

```bash
sudo systemctl status myapp.service --no-pager -l
sudo journalctl -u myapp.service -n 100 --no-pager
```

## Important detail

systemd captures what the program writes to stdout and stderr.

If the program opens and writes directly to its own file, systemd does not move that file's messages into the journal automatically.

---

# 11. How to Choose Where Service Logs Go

You control captured stdout and stderr using:

```ini
StandardOutput=
StandardError=
```

## Option 1: Send both to the journal

```ini
[Service]
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
```

View them with:

```bash
sudo journalctl -u myapp.service -f
```

This is the recommended starting point for most services.

Benefits include:

- Easy filtering by service
- Time-based filtering
- Automatic service name and PID information
- Automatic journal size management
- No separate logrotate rule for normal journal storage

---

## Option 2: Send stdout and stderr to separate files

```ini
[Service]
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log
```

The result is:

```text
stdout ---> /var/log/myapp/output.log
stderr ---> /var/log/myapp/error.log
```

Use `append:` for log files so that new messages are added to the existing files.

Follow the files:

```bash
sudo tail -f /var/log/myapp/output.log
```

```bash
sudo tail -f /var/log/myapp/error.log
```

---

## Option 3: Send both to one file

```ini
[Service]
StandardOutput=append:/var/log/myapp/myapp.log
StandardError=append:/var/log/myapp/myapp.log
```

Both streams go to:

```text
/var/log/myapp/myapp.log
```

The disadvantage is that normal messages and error messages are mixed together.

---

## Option 4: Send stdout to the journal and errors to a file

```ini
[Service]
StandardOutput=journal
StandardError=append:/var/log/myapp/error.log
```

The result is:

```text
stdout ---> systemd journal
stderr ---> /var/log/myapp/error.log
```

---

## Option 5: Discard output

```ini
[Service]
StandardOutput=null
StandardError=null
```

This discards both streams.

This is normally not recommended because you lose useful troubleshooting information.

---

# 12. Create a Log Directory with systemd

systemd can create a managed log directory for your service.

Use:

```ini
LogsDirectory=myapp
```

This normally creates:

```text
/var/log/myapp/
```

Example:

```ini
[Service]
User=myapp
Group=myapp
LogsDirectory=myapp
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log
```

This is cleaner than manually creating the directory and fixing its ownership.

Check the directory:

```bash
ls -ld /var/log/myapp
```

---

# 13. Complete Service with Separate Log Files

```ini
[Unit]
Description=My Python Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
LogsDirectory=myapp
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log

[Install]
WantedBy=multi-user.target
```

After saving the file, run:

```bash
sudo systemd-analyze verify /etc/systemd/system/myapp.service
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

Check the service:

```bash
sudo systemctl status myapp.service
```

Watch normal output:

```bash
sudo tail -f /var/log/myapp/output.log
```

Watch errors:

```bash
sudo tail -f /var/log/myapp/error.log
```

---

# 14. Application Logging versus systemd Logging

There are two different ways logs can be created.

## systemd stream logging

systemd handles messages that the process writes to:

```text
stdout
stderr
```

These are controlled by:

```ini
StandardOutput=
StandardError=
```

## Application-managed logging

An application can open and write directly to its own file.

Examples:

```text
Python logger  ---> /var/log/myapp/application.log
Java Logback   ---> /var/log/myapp/application.log
Nginx          ---> /var/log/nginx/access.log
```

The application's own configuration controls these files.

A useful design is:

```text
Application's detailed logs ---> application log file
Unexpected stderr           ---> systemd journal
Service start and stop       ---> systemd journal
```

The most important rule is:

> systemd controls the destination of stdout and stderr. The application controls files that it opens and writes to directly.

---

# 15. Why Python Logs Might Not Appear Immediately

Some programs buffer their output. Buffering means they wait before actually sending messages.

For Python, use unbuffered mode:

```ini
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
```

You can also flush a Python message immediately:

```python
print("Application started", flush=True)
```

Possible reasons for missing logs include:

- The application writes to its own file instead of stdout or stderr.
- The application does not produce any output.
- Output is buffered.
- The wrong service name was used with `journalctl`.
- A permission error prevents writing to a log file.
- The service failed before the application started.

---

# 16. Log Rotation for Custom Log Files

A log file can grow until it fills the disk. If you use files under `/var/log`, configure log rotation.

Create:

```text
/etc/logrotate.d/myapp
```

Example configuration:

```text
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

This means:

- `daily`: Check for rotation each day.
- `rotate 14`: Keep 14 old versions.
- `compress`: Compress old versions.
- `delaycompress`: Compress from the second old file onward.
- `missingok`: Do not fail if a log is missing.
- `notifempty`: Do not rotate an empty log.
- `copytruncate`: Copy the old content, then empty the active file.

Test the configuration without changing files:

```bash
sudo logrotate --debug /etc/logrotate.d/myapp
```

A forced rotation can be tested with:

```bash
sudo logrotate --force /etc/logrotate.d/myapp
```

Use forced rotation carefully on production systems.

---

# 17. Managing systemd Journal Size

Check journal disk usage:

```bash
sudo journalctl --disk-usage
```

The main journal configuration file is:

```text
/etc/systemd/journald.conf
```

Example:

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
SystemKeepFree=2G
MaxRetentionSec=14day
```

After changing it:

```bash
sudo systemctl restart systemd-journald
```

Persistent journals are normally stored in:

```text
/var/log/journal/
```

Temporary journals are normally stored in:

```text
/run/log/journal/
```

Clean old journal data by size:

```bash
sudo journalctl --vacuum-size=1G
```

Or by age:

```bash
sudo journalctl --vacuum-time=14d
```

Do not manually delete active journal files.

---

# 18. Safely Change an Installed Service

Do not directly edit package service files. Create an override:

```bash
sudo systemctl edit myapp.service
```

Example override:

```ini
[Service]
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log
```

Apply the change:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
```

See the original unit and all overrides:

```bash
sudo systemctl cat myapp.service
```

Overrides are normally stored under:

```text
/etc/systemd/system/myapp.service.d/override.conf
```

Check effective logging settings:

```bash
sudo systemctl show myapp.service \
  -p StandardOutput \
  -p StandardError
```

---

# 19. What Is Nginx?

**Nginx** is a web server and reverse proxy.

It can:

- Receive HTTP and HTTPS requests.
- Serve static files.
- Send requests to an application server.
- Handle TLS certificates.
- Load-balance traffic.
- Add request limits and security headers.
- Cache responses.

A common design is:

```text
Browser ---> Nginx ---> Application
```

Example:

```text
Browser ---> port 443 ---> Nginx ---> 127.0.0.1:8000 ---> Python app
```

Nginx itself is normally managed by systemd:

```bash
sudo systemctl status nginx.service
```

---

# 20. What Are Nginx Logs?

Nginx normally has two important logs:

1. Access log
2. Error log

Typical locations are:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

The exact paths are configurable.

## Nginx access log

The access log records incoming HTTP requests.

Example:

```text
192.168.1.10 - - [18/Aug/2026:10:30:04 +0530] "GET /health HTTP/1.1" 200 21 "-" "curl/8.5.0"
```

It can show:

- Client IP address
- Request date and time
- HTTP method
- Requested path
- HTTP status code
- Response size
- User agent

Follow it live:

```bash
sudo tail -f /var/log/nginx/access.log
```

## Nginx error log

The error log records Nginx problems and proxy problems.

Example:

```text
connect() failed (111: Connection refused) while connecting to upstream
```

This can mean:

- The backend service is stopped.
- The backend application crashed.
- Nginx uses the wrong IP address or port.
- Nothing is listening on the backend port.

Follow it live:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

# 21. Nginx Logs versus systemd Logs

## Nginx access log

Use it to answer:

- Did the request reach Nginx?
- Which path was requested?
- What status code was returned?
- Which client made the request?

Typical location:

```text
/var/log/nginx/access.log
```

## Nginx error log

Use it to answer:

- Could Nginx contact the backend?
- Was there a timeout?
- Was there a permission problem?
- Was an Nginx file missing?

Typical location:

```text
/var/log/nginx/error.log
```

## Application systemd journal

Use it to answer:

- Did the application start?
- Did the process crash?
- What did the application print?
- Was `ExecStart=` wrong?
- Did systemd restart the application?

Command:

```bash
sudo journalctl -u myapp.service
```

## Nginx systemd journal

Nginx also has systemd service logs:

```bash
sudo journalctl -u nginx.service
```

These commonly show:

- Nginx service start and stop events
- Nginx startup failures
- Configuration test or process errors
- Messages sent by Nginx to stdout or stderr

Individual web requests normally appear in the Nginx access log, not in `journalctl -u nginx`, unless Nginx logging has been specially configured.

## Easy way to remember

```text
Nginx access.log:
Who requested what, and what HTTP status was returned?

Nginx error.log:
What went wrong in Nginx or while contacting the backend?

journalctl -u myapp:
What happened inside the application or its service lifecycle?

journalctl -u nginx:
What happened to the Nginx service from systemd's view?
```

---

# 22. Complete Example: Python App, systemd, and Nginx

Assume:

```text
Application directory: /opt/myapp
Application user:      myapp
Backend address:       127.0.0.1:8000
Public domain:         example.com
Service name:          myapp.service
```

## Step 1: Create the application user

```bash
sudo useradd --system \
  --home /opt/myapp \
  --shell /usr/sbin/nologin \
  myapp
```

## Step 2: Create the application directory

```bash
sudo mkdir -p /opt/myapp
sudo chown myapp:myapp /opt/myapp
```

## Step 3: Create a simple Python application

Save this as `/opt/myapp/app.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
import sys


class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            body = json.dumps({"status": "healthy"}).encode()
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
            print("Health check completed", flush=True)
            return

        body = json.dumps({"message": "Hello from my application"}).encode()
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)
        print(f"Request received for {self.path}", flush=True)


try:
    server = HTTPServer(("127.0.0.1", 8000), Handler)
    print("Application listening on 127.0.0.1:8000", flush=True)
    server.serve_forever()
except Exception as error:
    print(f"Application failed: {error}", file=sys.stderr, flush=True)
    raise
```

Set ownership:

```bash
sudo chown myapp:myapp /opt/myapp/app.py
sudo chmod 750 /opt/myapp/app.py
```

## Step 4: Create the service

Create:

```text
/etc/systemd/system/myapp.service
```

Use:

```ini
[Unit]
Description=Example Python Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 -u /opt/myapp/app.py
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

Validate and start it:

```bash
sudo systemd-analyze verify /etc/systemd/system/myapp.service
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

Check its status:

```bash
sudo systemctl status myapp.service
```

Check port 8000:

```bash
sudo ss -lntp | grep ':8000'
```

Test the application directly:

```bash
curl http://127.0.0.1:8000/health
```

Expected response:

```json
{"status": "healthy"}
```

View application logs:

```bash
sudo journalctl -u myapp.service -f
```

## Step 5: Configure Nginx

On Ubuntu or Debian, create:

```text
/etc/nginx/sites-available/myapp
```

Use:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name example.com;

    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;

    location / {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 5s;
        proxy_read_timeout 60s;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/myapp \
  /etc/nginx/sites-enabled/myapp
```

Test Nginx configuration:

```bash
sudo nginx -t
```

If the test is successful, reload Nginx:

```bash
sudo systemctl reload nginx.service
```

Test through Nginx:

```bash
curl -H "Host: example.com" http://127.0.0.1/health
```

---

# 23. What Happens When a Request Arrives?

When a user opens:

```text
http://example.com/health
```

this happens:

1. The browser connects to Nginx on port 80.
2. Nginx receives the HTTP request.
3. Nginx writes request information to `myapp_access.log`.
4. Nginx forwards the request to `127.0.0.1:8000`.
5. The Python application processes the request.
6. The application writes `Health check completed` to stdout.
7. systemd sends that stdout message to the journal.
8. The application returns HTTP status 200.
9. Nginx returns the response to the browser.
10. Nginx records status 200 in its access log.

The request flow is:

```text
Client
  |
  | HTTP request
  v
Nginx service
  |
  | proxy_pass
  v
127.0.0.1:8000
  |
  v
myapp.service
  |
  v
Python process
```

The log flow is:

```text
HTTP request details ----------> Nginx access log
Nginx or proxy problem --------> Nginx error log
Application stdout/stderr -----> systemd journal
Service start/stop/failure ----> systemd journal
```

---

# 24. Troubleshooting Examples

## Problem: Nginx returns `502 Bad Gateway`

Check the Nginx error log:

```bash
sudo tail -n 100 /var/log/nginx/myapp_error.log
```

Then check the application:

```bash
sudo systemctl status myapp.service --no-pager -l
sudo journalctl -u myapp.service -n 100 --no-pager
sudo ss -lntp | grep ':8000'
```

Possible causes:

- The application service is stopped.
- The application crashed.
- Nginx uses the wrong port.
- The application listens on a different IP address.
- A firewall or permission rule blocks the connection.

## Problem: Service does not start

Check:

```bash
sudo systemctl status myapp.service --no-pager -l
sudo journalctl -u myapp.service -n 100 --no-pager
```

Common causes:

- `ExecStart=` points to a missing file.
- The Python path is wrong.
- The service user cannot read the application files.
- The working directory does not exist.
- Another process already uses the port.

Test the command as the service user:

```bash
sudo -u myapp /usr/bin/python3 -u /opt/myapp/app.py
```

## Problem: Nginx configuration is invalid

Test it:

```bash
sudo nginx -t
```

Check the Nginx service journal:

```bash
sudo journalctl -u nginx.service -n 100 --no-pager
```

Do not reload Nginx until `nginx -t` reports success.

## Problem: No application logs appear

Check:

```bash
sudo systemctl show myapp.service \
  -p StandardOutput \
  -p StandardError
```

Also check:

- Is the application printing anything?
- Is output buffered?
- Is the application writing directly to another file?
- Are you using the correct service name?
- Does the service have permission to write to the chosen file?

---

# 25. Recommended Logging Choices

## Choice A: Best simple starting point

Use the journal:

```ini
[Service]
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
```

Read logs with:

```bash
sudo journalctl -u myapp.service -f
```

This is easiest for most applications.

## Choice B: Separate output and error files

Use:

```ini
[Service]
LogsDirectory=myapp
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log
```

Also create a logrotate rule.

## Choice C: Application-managed logs

Use the application's logging framework when you need:

- `DEBUG`, `INFO`, `WARNING`, and `ERROR` levels
- JSON logs
- Request IDs
- Audit logs
- Application-controlled rotation

Keep stderr and service lifecycle messages available in the journal.

---

# 26. Quick Command Reference

## Application service

```bash
sudo systemctl status myapp.service
sudo systemctl restart myapp.service
sudo journalctl -u myapp.service -f
```

## Nginx service

```bash
sudo nginx -t
sudo systemctl status nginx.service
sudo systemctl reload nginx.service
sudo journalctl -u nginx.service -f
```

## Nginx request logs

```bash
sudo tail -f /var/log/nginx/myapp_access.log
sudo tail -f /var/log/nginx/myapp_error.log
```

## Network ports

```bash
sudo ss -lntp
```

## End-to-end tests

Test the application directly:

```bash
curl http://127.0.0.1:8000/health
```

Test through Nginx:

```bash
curl -H "Host: example.com" http://127.0.0.1/health
```

---

# 27. Final Summary

- A **process** is a running program.
- A **service** is usually a long-running background function.
- **systemd** starts, stops, restarts, and monitors services.
- A `.service` file tells systemd how to run an application.
- **stdin** is input going into a process.
- **stdout** is normal output from a process.
- **stderr** is error or diagnostic output from a process.
- A new systemd service's stdout and stderr normally go to the **systemd journal**.
- Read journal logs with `journalctl -u service-name.service`.
- Use `StandardOutput=` and `StandardError=` to choose log destinations.
- If you use custom files, use `append:` and configure log rotation.
- The application may also write directly to its own log files.
- Nginx access logs record HTTP requests and responses.
- Nginx error logs record Nginx and backend connection problems.
- `journalctl -u myapp` shows application output and service lifecycle events.
- `journalctl -u nginx` shows Nginx service lifecycle and startup information.

The simplest recommended setup is:

```ini
[Service]
StandardOutput=journal
StandardError=journal
```

Then read the logs with:

```bash
sudo journalctl -u myapp.service -f
```
