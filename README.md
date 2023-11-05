# Expose Kind LoadBalancer to the Public
Kind is a great project for quickly building development Kubernetes clusters.

What if you want to use a LoadBalancer as a service type in Kind?

The answer is MetalLB! You can follow this document to install and use MetalLB on your Kind cluster:

[Installing MetalLB for Kind](https://kind.sigs.k8s.io/docs/user/loadbalancer/)

Now you can create services with the LoadBalancer type in Kind. But what if you want to expose this service to the public interface of the machine, allowing access from other pods, localhost, or even the internet?

Then this simple method will help you!

First, install `socat` on your machine:
```bash
# For Debian-based distros
sudo apt install socat

# For Redhat-based distros
sudo yum install socat
```

Then install this script on the machine as a service by running these commands:
```bash
wget https://raw.githubusercontent.com/aleskxyz/kind-exposer/main/kind-exposer -O kind-exposer && \
chmod +x kind-exposer && \
sudo mv kind-exposer /usr/local/bin/ && \
wget https://raw.githubusercontent.com/aleskxyz/kind-exposer/main/kind-exposer@.service -O kind-exposer@.service && \
sudo mv kind-exposer@.service /etc/systemd/system/ && \
sudo systemctl daemon-reload
```

Now you are ready to expose the LoadBalancer services to the public by running this command:
```bash
sudo systemctl enable --now kind-exposer@<LoadBalancer-IP>:<Service-Port>:<Public-Port>:<Protocol>
```

For example, imagine we want to expose ingress-nginx on the public interface over ports 8080 and 8443. We've already created a service with the type of LoadBalancer (ingress-nginx-controller) that listens on TCP ports 80 and 443 over 172.18.100.10:
```bash
root@kind:~# kubectl get services -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.8.253     172.18.100.10   80:31896/TCP,443:32117/TCP   9d
ingress-nginx-controller-admission   ClusterIP      10.96.198.109   <none>          443/TCP                      9d
```

You only need to start the `kind-exposer` service with these commands:
```bash
sudo systemctl enable --now kind-exposer@172.18.100.10:80:8080:tcp
sudo systemctl enable --now kind-exposer@172.18.100.10:443:8443:tcp
```

And you can disable it by running these commands:
```bash
sudo systemctl disable --now kind-exposer@172.18.100.10:80:8080:tcp
sudo systemctl disable --now kind-exposer@172.18.100.10:443:8443:tcp
```

Don't forget to leave a star if it was helpful! 😉
