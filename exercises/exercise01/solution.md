# Exercise 1

## Create and run a web server with a custom page in HA

    echo "<h1>Hello Kube.camp</h1>" >> index.html
    echo "FROM nginx:alpine" >> Dockerfile
    echo "COPY index.html /usr/share/nginx/html" >> Dockerfile
    docker build -t ipedrazas/hello:kubecamp .
    docker push ipedrazas/hello:kubecamp

    kubectl run hello --image=ipedrazas/hello:kubecamp --expose --port=80 --replicas 2
