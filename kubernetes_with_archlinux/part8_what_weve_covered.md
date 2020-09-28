# Part 8: What we've covered so far...

At this point, I want to go over what we've covered so far in our quest to learn Kubernetes:

- In part 1, I've setup my deveopment environment and explored a few tools like Minikube, Visual Studo Code and k9s.
- In part 2, I've deployed my first Kubernetes application using a deployment and an NGINX ingress controller. There was no code involved, just NGINX and an HTML page. I learned the importance of building docker images using the docker daemon in Minikube. I installed the Kubernetes plugin for Visual Studio Code in an attempt to manage my deployments. After realizing that my deployment wasn't going to be accessible from outside the container without some help of an additional service, I installed the NGINX Ingress Controller on Minikube and created a ClusterIP service to act as a bridge. 
- In part 3, I wrote a REST service using Go, which I felt was much easier due to my familiarity with the language. I again used a ClusterIP service and the NGINX Ingress Controller. This was significantly easier to develop but I still don't have a handle on the meaning behind the ports used.
- In part 4, I took a small break from development and looked at some additional tools for Kubernetes. I found k9s to be the most useful and I still use it; however, it's can still be a challenge to use.
- In part 5, I added logging to my previous application just to see if it would make my application more interesting. Maybe I can make use of this later.
- In part 6, I created a REST service using Python and this time used a LoadBalancer (this was being used in an example on the Kubernetes website). However to access my application now I needed to use Minikube's tunnel service.
- In part 7, I created a deployment using Python and MySQL. This section was significantly harder than my previous deployments. I explored ConfigMaps and Secrets. I learned the importance of figuring out how to create the schema early on. I used docker-compose to create the initial application, but still found it significantly difficult to migrate it to Kubernetes. I wished that I could have gone straight to Kubernetes to save some time, and felt that maybe this came with more experience.

Time for a break.