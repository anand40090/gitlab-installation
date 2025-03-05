# Gitlab-installation
This is to install gitlab on local system / ubuntu system

# Update system packages
sudo apt update && sudo apt upgrade -y

# Stop and remove existing GitLab and GitLab Runner containers
sudo docker stop gitlab gitlab-runner
sudo docker rm gitlab gitlab-runner

# Pull the latest GitLab and GitLab Runner images
sudo docker pull gitlab/gitlab-ce:latest
sudo docker pull gitlab/gitlab-runner:latest

# Restart GitLab with updated image

```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest

```

# Restart GitLab Runner with updated image

```
docker run -d --name gitlab-runner --network host \
  --volume /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest

```
# Register gitlab runner 

```
docker exec -it gitlab-runner gitlab-runner register

You'll need the following information to register the runner:

GitLab URL: The address of your GitLab instance (e.g., http://gitlab.example.com).
GitLab registration token: Obtain it from your GitLab project under Settings > CI/CD > Runners.
Description: The name/description of the runner.
Tags: Optional tags to associate with the runner.
Executor: Choose an executor (e.g., docker).

```

# Restart Docker service (optional, if any issues)
sudo systemctl restart docker

# Verify that the containers are running
sudo docker ps -a

# Wait for GitLab to initialize (may take a few minutes)
echo "Waiting for GitLab to initialize..."
sleep 60  # Adjust sleep time if needed

# Retrieve GitLab root user password
```
if [ -f "/srv/gitlab/config/initial_root_password" ]; then
    echo "GitLab Root Login Credentials:"
    echo "Username: root"
    echo "Password: $(sudo cat /srv/gitlab/config/initial_root_password | grep Password | awk '{print $2}')"
else
    echo "Password file not found! Resetting root password..."
    sudo docker exec -it gitlab gitlab-rails console <<EOF
user = User.find_by_username('root')
user.password = 'NewSecurePassword'
user.password_confirmation = 'NewSecurePassword'
user.save!
exit
EOF
    echo "Password reset to: NewSecurePassword"
fi


```

# Display GitLab login URL
echo "Access GitLab at: http://$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)/"
/### Replcae the IP address from above

