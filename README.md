# Linux Server Configuration

In this project a Linux distribution was installed in a virtual machine in order to host a a web application.
The focus is to teach the students how to secure and set up a Linux server.


## Requirements

### Server Functionality

- Installing updates
- Security from attacks
- Database server configured to serve data
- All system packages have to be updated to most recent versions
- The web server responds on port 80.
- Web server has to be configured to serve the Item Catalog application as a WSGI app.

### User Management

- The SSH key can be used to log in as grader on the server
- You can not logging as root remotely
- Grader user has sudo access to inspect files that are readable only by root

### Security

- The firewall should be configurated to only allow SSH, HTTP and NTP
- Users are enforced to authenticate using RSA keys
- SSH is hosted on non-default port


## How to run the project

To access the server you have to use:
- IP address: 34.244.190.209
- SSH port:
- URL for the web application:
-

## Resources Used

- Amazon Lightsail

