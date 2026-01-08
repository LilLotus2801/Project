Install WSL in windows wsl --install
installing K3 
url -sfL https://get.k3s.io | sh - 
# Check for Ready node, takes ~30 seconds 
sudo k3s kubectl get node 

sudo systemctl status k3s for status check

sudo chmod 777 /etc/rancher/k3s/k3s.yaml 

kubectl get pods -A


install docker

lsb_release -a for ubuntu version

for installing docker in ubuntu
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update


sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

check status of docker
sudo systemctl status docker

sudo systemctl start docker

enable at boot
sudo systemctl enable docker

sudo chmod 777 /var/run/docker.sock && docker pull nginx:1.29.4

docker file
FROM nginx
COPY static-html-directory /usr/share/nginx/html


docker build -t hello-world-nginx:v1 .

docker images 

docker run --name hello-world-nginx hello-world-nginx:v1

docker run --name hello-world-nginx -d -p 8080:80 hello-world-nginx:v1
8080 host port 80 container port

kubectl run hello --image=hello-world-nginx:v1

sudo k3s ctr images list

docker save hello-world-nginx:v1 -o hello-world-nginx.tar

sudo k3s ctr images import hello-world-nginx.tar

sudo k3s ctr images list | grep hello-world-nginx

kubectl run hello2 --image=hello-world-nginx:v1

kubectl describe po hello2

imperative vs declartive
kubectl run hello --image=hello-world-nginx:v1(ex of imperative)
yml file creation of pods (ex of declarative)

kubectl get service

kubectl expose pod hello2 --name=hello2-service  --port=80 --target-port=80 --type=NodePort

 to get node ip
kubectl get nodes -o wide

curl http://172.21.84.48:30717

curl http://node-ip:exposed port

 kubectl api-resources 
 to get all the api resources

 after we create deplyment ands service yaml files

 kubectl apply -f deployment.yaml

  kubectl apply -f service.yaml
   kubectl apply -f service.yaml -f deployment.yaml

kubectl apply -f test (directory containing yaml files)

kubectl get pods,replicaset,deployment,service
kubectl get po,rs,deploy,svc

kubectl get nodes -o wide
curl 172.21.84.48:31494


