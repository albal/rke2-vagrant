# RKE2 through Vagrant

```bash
vagrant init
vagrant up
vagrant ssh master
kubectl get nodes -o wide
kubectl run nginx --image nginx
kubectl expose pod nginx --port 80
kubectl get svc
curl http://<cluster-ip>
```
