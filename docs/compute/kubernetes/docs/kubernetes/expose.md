---
layout: article
title: Kubectl
permalink: /docs/kubectl-expose.html
key: kubectl-expose
aside:
  toc: true
sidebar:
  nav: docs
---
## Exposing Applications

Basically, there are two kinds of exposable applications: *web-based* applications, *other* applications. The difference between these types is in required IP address. The Web-based application shares IP address with other Web-based applications while the other applications require new IP address for each service. Number of IP addresses is limited so if possible, use the Web-based approach.

In the following documentation, there are YAML fragments, they are meant to be deployed using the `kubectl` like this: `kubectl create -f file.yaml -n namespace` where `file.yaml` contains those YAMLs below and `namespace` is the namespace with the application. Furthermore, this part of documentation suppose, that the application (deployment) already exists. If not sure, read again [hello example](/containers/get-started/hello-world/).

### Web-based Applications

Web-based applications are those that communicate via `HTTP` protocol. These applications are exposed using `Ingress` rules. 

This kind of applications require a service that binds a port with application and ingress rules that expose the service to the Internet.

Suppose, we have an application that is ready to serve on port 8080. We create a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: application-svc
spec:
  type: ClusterIP
  ports:
  - name: application-port
    port: 80
    targetPort: 8080
  selector:
    app: application
```

Where `selector:` `app: application` must match application name in deployment and `application-svc` and `application-port` are arbitrary names.

Once we have a service, we create ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: application-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - "application.dyn.cloud.e-infra.cz"
      secretName: application-dyn-clout-e-infra-cz-tls
  rules:
  - host: "application.dyn.cloud.e-infra.cz"
    http:
      paths:
      - backend:
          service:
            name: application-svc
            port:
              number: 80
        pathType: ImplementationSpecific
```

Where `service` `name: application-svc` must match metadata name from the *Service*. This *Ingress* exposes the application on web URL `application.dyn.cloud.e-infra.cz`. Note: domain name is automatically registered if it is in `dyn.cloud.e-infra.cz` domain. The `tls` section creates Lets Encrypt certificate, no need to maintain it. 

If `tls` section is used, TLS is terminated at system NGINX Ingress. Application must be set to communicate via `HTTP`. Communication between Kubernetes and a user is via `HTTPS`.

*IMPORTANT* 
TLS is terminated in NGINX Ingress at cluster boundary. Communication inside cluster is not encrypted, mainly not inside a single cluster node. If this is a problem, user needs to omit `tls` section and `anotations` section and provide a certificate and key on his/her own to the Pod. 

*IMPORTANT*
Some applications are confused that they are configured as `HTTP` but exposed as `HTTPS`. If an application generates absolute URLs, it must generate `HTTPS` URLs and not `HTTP`. Kubernetes Ingress sets `HTTP_X_SCHEME` header to `HTTPS` if it is TLS terminated. E.g., for *django*, user must set: `SECURE_PROXY_SSL_HEADER = ("HTTP_X_SCHEME", "https")`. Common header used for this situation is `X_FORWARDED_PROTO` but this is not set by Kubernetes NGINX. It is possible to expose also 'HTTPS' configured applications. See below *HTTPS Target*.

#### Authentication

`Ingress` can request user authentication and thus provide restricted access to the `Service`. Authentication can be set using a `Secret` and annotations.

First, setup a secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secretref
type: Opaque
data:
   auth: password
```

Where `secretref` is arbitrary name and password is `base64` encoded `htpasswd`. Password can be generated by this command: `htpasswd -n username | base64 -w0` where `username` is desired login username. It outputs text like `Zm9vOiRhcHIxJGhOZVRzdngvJEkyVk9NbEhHZDE1N1gvYTN2aktYSDEKCg==` which shall be put instead of the `password` into the secret above.

The following two annotations are required in the `Ingress`. 

```yaml
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: secretref
```

The `secretref` must match tne metadata name of the `Secret`.

*IMPORTANT* 
The password protection is applied only on external traffic, i.e., user will be prompted for a password. However, traffic from other pods bypasses authentication if it communicates directly with the `Service` IP. This can be mitigated applying `Network Policy`. See [Security](/containers/kubernetes/security/).

#### Big Data Upload

If big data upload is expected, the following two `Ingress` annotations might be necessary to deal with upload size limit.
```yaml
nginx.ingress.kubernetes.io/proxy-body-size: "600m"
nginx.org/client-max-body-size: "600m"
```

Replace the `600m` value with desired max value for upload data.

#### HTTPS Target

Ingress object can expose applications use HTTPS procol instead of HTTP, i.e., communication between ingress and application is encrypted as well. In this case, you need to add `nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"` annotation to the Ingress object.

### Other Applications

Applications that to not use `HTTP` protocol are exposed directly via `Service`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: application-svc
  annotations:
    external-dns.alpha.kubernetes.io/hostname: application.dyn.cloud.e-infra.cz
spec:
  type: LoadBalancer
  ports:
  - port: 22
    targetPort: 2222
  selector:
    app: application
```

Where `selector:` `app: application` must match application name in deployment and `application-svc` is arbitrary name. If annotation `external-dns.alpha.kubernetes.io/hostname` is provided and value i in domain `dyn.cloud.e-infra.cz`, the name is registered in DNS. 

Deploying this *Service* exposes the application on a public IP. One can check the IP with `kubectl get svc -n namespace` where `namespace` is namespace of the Service and application.

It is also possible to expose application on MUNI private IP. In this case, the application will be reachable from MUNI network or using MUNI VPN. This type of exposing is selected using annotation `purelb.io/service-group: privmuni`. This case is preferred as it does not consume public IP. Full example is below.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: application-svc
  annotations:
    purelb.io/service-group: privmuni
    external-dns.alpha.kubernetes.io/hostname: application.dyn.cloud.e-infra.cz
spec:
  type: LoadBalancer
  ports:
  - port: 22
    targetPort: 2222
  selector:
    app: application
```
