# Exercise 02

Now that you have deployed a web server, let's interact with it.


First, let's get the pods:

        kubectl get pods

now, let's read the logs of that pod:

        kubectl get logs hello-1767487725-smthc

Let's send some traffic... so, how do we access the pod? a solution is to bind the
pod port to our localhost:

        kubectl port-forward hello-1767487725-smthc 8000:80

Let's try to access our website:

        curl localhost:8000

let's read the logs again:

        kubectl get logs hello-1767487725-smthc

Can we tail the logs?

        kubectl get logs hello-1767487725-smthc -f

 or

        kubectl get logs -f hello-1767487725-smthc



