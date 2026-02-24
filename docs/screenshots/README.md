# Screenshot Checklist

Place the requested project evidence screenshots in this folder.

Required files:

- `01-github-actions-workflow.png`
- `02-github-actions-success-run.png`
- `03-dockerhub-images.png`
- `04-ec2-deployment-containers.png`
- `05-app-ui-home.png`
- `06-nginx-config-and-routing.png`

Suggested capture points:

- GitHub Actions workflow YAML and pipeline graph
- Successful CI/CD run with build and deploy jobs green
- Docker Hub repositories/tags for backend and frontend
- EC2 terminal output of `docker compose -f docker-compose.prod.yml ps`
- Browser screenshot of app running on `http://<EC2_PUBLIC_IP>`
- Nginx config (`frontend/nginx.conf`) and architecture/traffic flow evidence
