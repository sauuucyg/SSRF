# Flask Vulnerable App: Setup and Exploitation Guide

This guide walks you through setting up a Server-Side Request Forgery (SSRF) attack.

---

## Prerequisites

1. **Docker Installed**: Ensure Docker is installed on your system.
   - Check Docker version:
     ```bash
     docker --version
     ```
2. **Basic Command Line Knowledge**: Ability to navigate and run commands in the terminal.

---

## Setting Up the Application
### Step 1: Clone the following github repo


There is a Dockerfile in this repo that will spin up a vulnerable docker image. To build it, simply run docker build -t <tag> .. The Apache server is running on port 80 inside the container. Expose it with the -p flag. Running it with docker run -p 9000:80 <tag> will bring up a container listening on localhost:9000.



## Exploiting the Vulnerability

### Payload for Exploitation
1. Application code that fetches and display the content of the specified file

In programming languages, there are functions which can fetch the contents of locally saved file. These functions may be capable of fetching the content from remote URLs as well local files (e.g file_get_contents in PHP).

This functionality can be abused if application is not prepending any string to the user supplied data to fetch the content from a file i.e application is not prepeding and directory name or path to the user supplied data.

In this case, these data fetching function can process the schemes like "http://" or "file://". When user specifies the remote URL in place of file name like "http://localhost", the data fetching function extract the data from the specified URL.

In case if application is prepending any data string (for example any directory name) to user data, "http://" or "file://" scheme won't work and exploitation of SSRF vulnerability is not possible.

2. Application provides interface to connect to Remote Host

Web application has interfaces that allow an user to specify the any IP with any port. Here the application has functionality which tries to connect to service like "MySQL", "LDAP" etc.

Application expects user to specify the remote server hostname/IP, username and password in input fields. Application then tries to connect to the remote server over specified port. Here in this scenario, application tries to communicate to remote service listening on specific port. When vulnerable code has functionality to connect to server like MySQL and user specified the SMB port, vulnerable application will try to communicate to SMB servie using MySQL server service packets. Even though, the port is open, we are not able to communicate to the service due to difference in way of communication.

This behaviour can be exploited to perform internal network scanning not just to enumerate IPs but Ports as well on those live IPs.

3. Application with File Download Functionality

In this case, an attacker can exploit this functionality to perform IP scanning inside the network where application server is hosted. The function which performs the task of downloading file from server, can download file not just from local server but also from SMB path as well. This is something which can help an attacker to figure out the Windows based machines in the network.

Web application hosted on Windows OS will process the SMB path as well if file download functionality is processing user input without prepending any data.

4. Bypassing IP blacklisting using DNS Based Spoofing

The script has funcionality which allow user to fetch data from remote URL. User need to specify the remote URL with any IP or domain name.

The script perform check if user has specified the input as "localhost", "Internal IPs" or "Reserved IPs". If domain/IP specified by user is blacklisted, script will not fetch the content and stop processing.

5. Bypassing IP blacklisting using DNS Rebinding Technique

Application has implemented black listing of not just internal and private range IPs but also rsolve the user supplied domain to its IP and again perform check if resolved is black listed or not.

In this case, DNS based spoofing trick will also not work to access the content hosted on internal/Reserved IP. Application code perform domain resolution to its IP and again perform black listed IP check for the resolved IP.

6. SSRF in HTML to PDF generator script

This the scenrio of the web app which is using HTML to PDF generator script and passing untrusted user supplied data to HTML file which is processed by HTML to PDF generator.

**Disclaimer**: This guide is intended solely for educational purposes within controlled environments. Unauthorized testing or exploitation of vulnerabilities is illegal and may result in severe consequences. Always obtain explicit permission before testing applications you do not own.
