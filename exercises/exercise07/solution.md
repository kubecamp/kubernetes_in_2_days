# Create self-signed Certificate

To create a self-signed certificate we need to execute teh following command:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs/tls.key -out certs/tls.crt

The command will ask a few questions, the important one is to put the right FQDN (we use lab.kube.camp), which it will be the domain we want to use with this certificate.

Modify your file `/etc/hosts` with

    127.0.0.1   lab.kube.camp

Once we have the certificate, we need to configure nginx. Create a file called `ssl-nginx.conf` with this content:

    server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        listen 443 ssl;

        root /usr/share/nginx/html;
        index index.html index.htm;

        server_name lab.kube.camp;
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;

        location / {
                try_files $uri $uri/ =404;
        }
    }



Try to run a docker container using these certificates and verify that it works

    docker run -d --rm -p 8443:443 -p 8000:80 -v $(pwd)/conf/ssl-nginx.conf:/etc/nginx/conf.d/default.conf:ro -v $(pwd)/certs:/etc/nginx/ssl nginx


Test it by doing

    curl http://lab.kube.camp:8000

and

    curl -k https://lab.kube.camp:8443


Now it's time to turn this into kubernetes manifests. We are going to create:

    * Secret: containing the certificates
    * ConfigMap: containing the nginx conf
    * Deployment: containing nginx image. We will mount the secret and configmap as two volumes


To create the secret you can create a `yaml` file or you can execute the following command:

    kubectl create secret tls nginx-ssl --cert=certs/tls.crt --key=certs/tls.key

To create the configmap, you can create it using the following command, or creating the file:

    kubectl create configmap nginx-config --from-file=conf/ssl-nginx.conf

Finally, you have to create a deployment for nginx and a service:

Service:

    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: nginx
      name: nginx
    spec:
      type: NodePort
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
        name: http
      - port: 443
        protocol: TCP
        targetPort: 443
        name: https
      selector:
        run: nginx

Deployment:

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      labels:
        run: nginx
      name: nginx
    spec:
      replicas: 1
      selector:
        matchLabels:
          run: nginx
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - image: nginx:alpine
            imagePullPolicy: Always
            name: nginx
            ports:
            - containerPort: 80
              protocol: TCP
            - containerPort: 443
              protocol: TCP
            volumeMounts:
            - name: nginx-certs
              readOnly: true
              mountPath: /etc/nginx/ssl
            - name: config-nginx
              mountPath: /etc/nginx/conf.d
          volumes:
            - name: nginx-certs
              secret:
                secretName: nginx-ssl
            - name: config-nginx
              configMap:
                name: nginx-config
                items:
                - key: ssl-nginx.conf
                  path: default.conf

To deploy, you can use `kubectl`

    kubectl create -f .

## Test your deployment

First, you need to know which ports your services have been used:

    kubectl get svc

You should see something like this:

    NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
    kubernetes   10.0.0.1     <none>        443/TCP                      3d
    nginx        10.0.0.9     <nodes>       80:31732/TCP,443:32269/TCP   4m

Note the `PORTS` column, it says that the port 80 of the service has been bound to the port 31732 of teh host. If you're using minikube to do this exercise, you need to get the IP of minikube:

    minikube ip

Set that ip in your file `/etc/hosts` as the domain you used to create the certificates (you had previously it set to 127.0.0.1 during the docker test).

Test it by doing

    curl http://lab.kube.camp:31732

and

    curl -k https://lab.kube.camp:32269

Excellent, time to scale:

    kubectl scale deployment nginx --scale=3

Congratulate yourself if you get 200 OK twice!
