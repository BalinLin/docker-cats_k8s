`kubernetes: v1.25.3` `Minicube: v1.28.0`

## Before running the project.

- Install kubernetes and [start minikube with Calico CNI Manifest](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube)
```bash
minikube start --network-plugin=cni
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/calico.yaml
```
- (Optional) Install [OpenLens](https://github.com/MuhammedKalkan/OpenLens) and [kubens](https://github.com/ahmetb/kubectx)

- (Optional) Create registry secret and `.yaml`
```bash
kubectl create secret docker-registry <your-secret-name> --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email> --output=yaml > <your-yaml-filename>
```

## Run the project.
1. Change the image path in `docker-cats.yaml` to your registry

2. Create a namespace call `docker-cats`
```bash
kubectl create namespace docker-cats
```

3. Change current namespace
```bash
kubens docker-cats
```

4. Enabel addons
```bash
minikube addons enable ingress # ingress
minikube addons enable metrics-server # hpa
```

5. Apply all the `.yaml`
```bash
kubectl apply -f docker-cats.yaml
kubectl apply -f docker-cats-configmap.yaml
kubectl apply -f docker-cats-ingress.yaml
kubectl apply -f docker-cats-network.yaml
```

6. Direct the ingress domain name to localhost in order to test ingress.
```bash
# You can also modify your `/etc/hosts` by `sudo vim`
sudo -- sh -c 'echo "127.0.0.1 baby.com" >> /etc/hosts'
sudo -- sh -c 'echo "127.0.0.1 green.com" >> /etc/hosts'
sudo -- sh -c 'echo "127.0.0.1 dark.com" >> /etc/hosts'
```

7. Run `minikube tunnel` to enable http and https port
```bash
# The service/ingress docker-cats-ingress requires privileged ports to be exposed: [80 443]
minikube tunnel
```

8. Open your browser and paste the following url
```bash
http://baby.com/
http://green.com/
http://dark.com/
```

9. Test for liveness probe
```bash
kubectl exec -it <pod name> -- sh

# Ref: https://stackoverflow.com/questions/11583562/how-to-kill-a-process-running-on-particular-port-in-linux
lsof -i:8080
kill <busybox PID>
```

10. Test for HPA
```bash
while true; do curl http://baby.com; done
```

11. Test network policy
- Try to modify `docker-cats-network.yaml` and apply it to see the difference.
