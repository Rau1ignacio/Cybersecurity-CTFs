# Borazuwarah CTF

## Description

This challenge consists of exploiting a vulnerable Docker application to gain access to a limited shell. The goal is to retrieve a flag located in a specific directory.

## Requirements

- Docker installed on your machine
- Basic knowledge of command-line interface

## Steps to Solve

1. **Pull the Docker Image**: Use the following command to pull the Docker image:
   ```bash
   docker pull vulnerable_docker_image
   ```

2. **Run the Docker Container**: Start the container with the command:
   ```bash
   docker run -d -p 8080:80 vulnerable_docker_image
   ```

3. **Access the Application**: Open your browser and go to `http://localhost:8080`

4. **Exploit the Vulnerability**: Analyze the application to find the vulnerability and exploit it.

5. **Retrieve the Flag**: Once you gain access to the shell, navigate to the desired directory and retrieve the flag using:
   ```bash
   cat /path/to/flag.txt
   ```

## Conclusion

With the completion of this challenge, you should have a good understanding of exploiting Docker containers.

## Additional Resources

- [Docker Documentation](https://docs.docker.com)
- [CTF Writeups](https://ctf-writeups.com)