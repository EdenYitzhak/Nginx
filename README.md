####
## Step-by-Step Process
### Step 2:Docker Containerization

Dockerize NGINX and configure it to respond with the text "yo this is nginx" upon access.
Dockerized NGINX can run locally on the private machine.

Process:
1. Run the following commands to pull and run the Docker image:

```hcl
docker pull edeny/my-nginx
docker run -d -p 80:80 edeny/my-nginx
Once the container is running, go to http://localhost to see the "yo this is nginx" message
```
![image](https://github.com/user-attachments/assets/5e8be404-f6e2-4ce8-a515-301123caad1c)
