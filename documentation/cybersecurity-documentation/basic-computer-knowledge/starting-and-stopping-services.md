# 🔛 Starting and Stopping Services

1. **Starting Services:**
   * `sudo service apache2 start`:
     * _Explanation:_ Starts the Apache web server service.
     * _Example:_ Running `sudo service apache2 start` would initiate the Apache web server and make it available for serving web pages.
   *   `python3 -m http.server 80`:

       * _Explanation:_ Starts a simple HTTP server using Python on port 80.
       * _Example:_ Running `python3 -m http.server 80` would start a basic HTTP server on port 80, allowing you to serve files from the current directory.

       <figure><img src="https://linuxhint.com/wp-content/uploads/2021/03/image9-20.png" alt=""><figcaption></figcaption></figure>
2. **Stopping Services:**
   * `sudo service apache2 stop`:
     * _Explanation:_ Stops the Apache web server service.
     * _Example:_ Running `sudo service apache2 stop` would halt the running Apache web server, shutting down any active web page serving.
3. **Enabling and Disabling Services on System Boot:**
   * `sudo systemctl enable ssh`:
     * _Explanation:_ Enables the SSH (Secure Shell) service to start automatically on system boot.
     * _Example:_ Running `sudo systemctl enable ssh` would configure the system to start the SSH service during system startup.
   * `sudo systemctl disable ssh`:
     * _Explanation:_ Disables the SSH service from starting automatically on system boot.
     * _Example:_ Running `sudo systemctl disable ssh` would prevent the SSH service from starting automatically during system startup.
4. **Checking the Status of Services:**
   * `sudo service apache2 status`:
     * _Explanation:_ Displays the current status of the Apache web server service.
     * _Example:_ Running `sudo service apache2 status` would show whether the Apache service is running, stopped, or encountering issues.
   * `sudo systemctl status ssh`:
     * _Explanation:_ Displays the current status of the SSH service.
     * _Example:_ Running `sudo systemctl status ssh` would provide information about the status of the SSH service, including whether it's active or inactive.

**Starting and Stopping Services:**

*   `sudo service apache2 restart`:

    * _Explanation:_ Restarts the Apache web server service.
    * _Example:_ Running `sudo service apache2 restart` would stop and then start the Apache web server, applying any configuration changes.

    <figure><img src="https://phoenixnap.com/kb/wp-content/uploads/2021/04/apache-status-active.png" alt=""><figcaption></figcaption></figure>
* `sudo systemctl start serviceName`:
  * _Explanation:_ Starts a service using systemd.
  * _Example:_ Running `sudo systemctl start apache2` would start the Apache web server using systemd.
* `sudo systemctl stop serviceName`:
  * _Explanation:_ Stops a service using systemd.
  * _Example:_ Running `sudo systemctl stop apache2` would stop the running Apache web server.

**Enabling and Disabling Services:**

* `sudo service apache2 enable`:
  * _Explanation:_ Enables the Apache web server service to start on boot.
  * _Example:_ Running `sudo service apache2 enable` would configure Apache to start automatically during system startup.

<figure><img src="https://phoenixnap.com/kb/wp-content/uploads/2021/04/stop-apache-status-inactive.png" alt=""><figcaption></figcaption></figure>

* `sudo service apache2 disable`:
  * _Explanation:_ Disables the Apache web server service from starting on boot.
  * _Example:_ Running `sudo service apache2 disable` would prevent Apache from starting automatically during system startup.
* `sudo systemctl is-enabled serviceName`:
  * _Explanation:_ Checks if a service is enabled to start on boot using systemd.
  * _Example:_ Running `sudo systemctl is-enabled apache2` would indicate whether Apache is set to start on boot.

**Checking Service Logs:**

* `journalctl -u serviceName`:
  * _Explanation:_ Displays the logs for a specific service.
  * _Example:_ Running `journalctl -u apache2` would show logs related to the Apache web server.

**Reloading Service Configuration:**

* `sudo systemctl reload serviceName`:
  * _Explanation:_ Reloads the configuration of a running service without stopping it.
  * _Example:_ Running `sudo systemctl reload apache2` would reload the configuration of the Apache web server.
