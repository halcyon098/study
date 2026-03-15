```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": 
  [
  "https://cf-workers-docker-io-ks3.pages.dev",
  "https://docker.registry.cyou",
  "https://docker-cf.registry.cyou",
  "https://docker.jsdelivr.fyi",
  "https://dockercf.jsdelivr.fyi",
  "https://dockertest.jsdelivr.fyi",
  "https://atomhub.openatom.cn"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/proxy.conf <<-'EOF'
[Service]
Environment="HTTP_PROXY=http://10.92.100.5:7897/"
Environment="HTTPS_PROXY=http://10.92.100.5:7897/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
博客：
https://fastly.blog.cmliussss.com/p/CF-Workers-docker.io/
