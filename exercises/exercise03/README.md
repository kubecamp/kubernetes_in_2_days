# Exercise 03

## Services

Binding our pod to our localhost can be great for development, but it's nbot going to
work very well in prod. Yes, `it works in my machine is as true as useless`.

Let's expose our pods to the world using a service:

        kubectl expose deployment hello

So, we have a service that sends traffic to our pods. Excellene, let's look at it:


        kubectl get svc

What about if we want to look at our service?

        kubectl describe svc hello

What if I want to access the pod from outside my cluster?

        kubectl expose deployment hello --name=public-hello --type=LoadBalancer

Or

        kubectl expose deployment hello --name=public-hello --type=NodePort

Next you have to access that service. To get the IP/port you will have to do:

	
	kubectl describe svc public-hello

If you're using `minikube` you need to get the ip of minikube:

	minikube ip

or you can ask minikube to open your service

	minikube service public-hello



