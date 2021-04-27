# portainer 

```shell
docker run -d --name portainer --restart=always -p 7084:9000 -v /var/run/docker.sock:/var/run/docker.sock -v /home/docker/portainer_data:/data portainer/portainer
```





