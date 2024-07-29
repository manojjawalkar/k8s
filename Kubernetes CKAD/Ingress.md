- Provides a single externally accessible URL
- This URL can be configured to route to different services (service/games, service/develop, service/payment etc) within the cluster
- Takes care of encryption, loadbalancing
- Layer7 LoadBalancer
- You will still need a nodeport to expose your service to external world, but it is just one time configuration unlike in a service
- What was disadvantag using a service?
    - If we have a nodeport exposed, we would need a NLB for the service ( this is automatically creted by the cloud like GCP)
    - We later decided to add more services, so created another NLB.
    - Now there are two NLBs so how do you send request to respective service
    - We need to add another LB which would redirect traffic based on url.
    - This is not easy to manage as the product grows
    - Hence we have ingress

# Working

- Two parts - Deploy & Configure
    - Deploy
        - K8s allows you to chose the supported solution out of GCE, NGINX, TRAEFIK, ISTIO, HAPROXY etc
        - GCE and NGINX are currently being supported and maintained by the Kubernetes project
            - We can use a deployment definition to deploy an NGNIX contoller
        - This is called **Ingress Controller**
        - This isn't created by default by k8s
        
    - Configure
        - Specify the set of rules to configure ingress
        - These rules are referred to as **Ingress Resources**
        - Resources can be defined using a definition file like any other object in k8s

# Imperative Way

```
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

* * *

# FAQ - What is the rewrite-target option?

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

Our watch app displays the video streaming webpage at http://<watch-service>:&lt;port&gt;/</watch-service>

Our wear app displays the apparel webpage at http://<wear-service>:&lt;port&gt;/</wear-service>

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:&lt;port&gt;/</watch-service></ingress-port></ingress-service>

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:&lt;port&gt;/</wear-service></ingress-port></ingress-service>

Without the rewrite-target option, this is what would happen:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:&lt;port&gt;/watch</watch-service></ingress-port></ingress-service>

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:&lt;port&gt;/wear</wear-service></ingress-port></ingress-service>

Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

For example: replace(path, rewrite-target)  
In our case: replace("/path","/")

# Reference:

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-
- https://kubernetes.io/docs/concepts/services-networking/ingress
- https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types