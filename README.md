<!DOCTYPE html>
<html>
<head>
  
</head>
<body>
  <h1>Securing NGINX-ingress</h1>
  <h2> DISCLAIMER:</h2> <br> 
  <code>This tutorial will detail how to install and secure ingress to your cluster using NGINX.</code>

  <h2>Step 1 - Install Helm</h2>
  <p>Skip this section if you have helm installed.</p>
  <p>The easiest way to install cert-manager is to use Helm, a templating and deployment tool for Kubernetes resources.</p>
  <p>First, ensure the Helm client is installed following the Helm installation instructions.</p>
  <p>For example, on MacOS:</p>
  <pre><code>brew install kubernetes-helm</code></pre>

  <h2>Step 2 - Deploy the NGINX Ingress Controller</h2>
  <p>A kubernetes ingress controller is designed to be the access point for HTTP and HTTPS traffic to the software running within your cluster. The ingress-nginx-controller does this by providing an HTTP proxy service supported by your cloud provider's load balancer.</p>
  <p>You can get more details about ingress-nginx and how it works from the documentation for ingress-nginx.</p>
  <p>Add the latest helm repository for the ingress-nginx</p>
  <pre><code>helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx</code></pre>
  <p>Update the helm repository with the latest charts:</p>
  <pre><code>$ helm repo update
  Hang tight while we grab the latest from your chart repositories...
  ...Skip local chart repository
  ...Successfully got an update from the "stable" chart repository
  ...Successfully got an update from the "ingress-nginx" chart repository
  ...Successfully got an update from the "coreos" chart repository
  Update Complete. ⎈ Happy Helming!⎈</code></pre>
  <p>Use helm to install an NGINX Ingress controller:</p>
  <pre><code>$ helm install quickstart ingress-nginx/ingress-nginx
  NAME: quickstart
  ... lots of output ...</code></pre>
  <p>It can take a minute or two for the cloud provider to provide and link a public IP address. When it is complete, you can see the external IP address using the kubectl command:</p>
  <pre><code>$ kubectl get svc
  NAME                                            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
  kubernetes                                      ClusterIP      10.0.0.1       &lt;none&gt;        443/TCP                      13m
  quickstart-ingress-nginx-controller             LoadBalancer   10.0.114.241   &lt;pending&gt;     80:31635/TCP,443:30062/TCP   8m16s
  quickstart-ingress-nginx-controller-admission   ClusterIP      10.0.188.24    &lt;none&gt;        443/TCP                      8m16s</code></pre>
  <p>This command shows you all the services in your cluster (in the default namespace), and any external IP addresses they have. When you first create the controller, your cloud provider won't have assigned and allocated an IP address through the LoadBalancer yet. Until it does, the external IP address for the service will be listed as &lt;pending&gt;.</p>
  <p>Your cloud provider may have options for reserving an IP address prior to creating the ingress controller and using that IP address rather than assigning an IP address from a pool. Read through the documentation from your cloud provider on how to arrange that.</p>

  <h2>Step 3 - Assign a DNS name</h2>
  <p>The external IP that is allocated to the ingress-controller is the IP to which all incoming traffic should be routed. To enable this, add it to a DNS zone you control, for example as www.example.com.</p>
  <p>This quick-start assumes you know how to assign a DNS entry to an IP address and will do so.</p>
  <br>
  <h1>Step 4 - Deploy an Example Service</h1>
  <p>Your service may have its own chart, or you may be deploying it directly with manifests. This quick-start uses manifests to create and expose a sample service. The example service uses kuard, a demo application.</p>
  <p>The quick-start example uses three manifests for the sample. The first two are a sample deployment and an associated service:</p>
  
  <pre><code>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard
spec:
  selector:
    matchLabels:
      app: kuard
  replicas: 1
  template:
    metadata:
      labels:
        app: kuard
    spec:
      containers:
      - image: gcr.io/kuar-demo/kuard-amd64:1
        imagePullPolicy: Always
        name: kuard
        ports:
        - containerPort: 8080

apiVersion: v1
kind: Service
metadata:
  name: kuard
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: kuard
  </code></pre>
  
  <p>You can create download and reference these files locally, or you can reference them from the GitHub source repository for this documentation. To install the example service from the tutorial files straight from GitHub, do the following:</p>
  
  <pre><code>kubectl apply -f https://raw.githubusercontent.com/cert-manager/website/master/content/docs/tutorials/acme/example/deployment.yaml
  # expected output: deployment.extensions "kuard" created

kubectl apply -f https://raw.githubusercontent.com/cert-manager/website/master/content/docs/tutorials/acme/example/service.yaml
  # expected output: service "kuard" created
  </code></pre>
  
  <p>An Ingress resource is what Kubernetes uses to expose this example service outside the cluster. You will need to download and modify the example manifest to reflect the domain that you own or control to complete this example.</p>
  
  <p>A sample ingress you can start with is:</p>
  
  <pre><code>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kuard
  annotations:
    #cert-manager.io/issuer: "letsencrypt-staging"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.example.com
    secretName: quickstart-example-tls
  rules:
  - host: example.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kuard
            port:
              number: 80
  </code></pre>
  
  <p>You can download the sample manifest from GitHub, edit it, and submit the manifest to Kubernetes with the command below. Edit the file in your editor, and once it is saved:</p>
  
  <pre><code>kubectl create --edit -f https://raw.githubusercontent.com/cert-manager/website/master/content/docs/tutorials/acme/example/ingress.yaml
  # expected output: ingress.networking.k8s.io/kuard created
  </code></pre>
  
  <p>Note: The ingress example we show above has a host definition within it. The ingress-nginx-controller will route traffic when the hostname requested matches the definition in the ingress. You can deploy an ingress without a host definition in the rule, but that pattern isn't usable with a TLS certificate, which expects a fully qualified domain name.</p>
  
  <p>Once it is deployed, you can use the command <code>kubectl get ingress</code> to see the status of the ingress:</p>
  
  <pre><code>
NAME      HOSTS     ADDRESS   PORTS     AGE
kuard     *                   80, 443   17s
  </code></pre>
  
  <p>It may take a few minutes, depending on your service provider, for the ingress to be fully created. When it has been created and linked into place, the ingress will show an address as well:</p>
  
  <pre><code>
NAME      HOSTS     ADDRESS         PORTS     AGE
kuard     *         203.0.113.2   80        9m
  </code></pre>
  
  <p>Note: The IP address on the ingress may not match the IP address that the ingress-nginx-controller has. This is fine, and is a quirk/implementation detail of the service provider hosting your Kubernetes cluster. Since we are using the ingress-nginx-controller instead of any cloud-provider specific ingress backend, use the IP address that was defined and allocated for the quickstart-ingress-nginx-controller LoadBalancer resource as the primary access point for your service.</p>
  
  <p>Make sure the service is reachable at the domain name you added above, for example http://www.example.com. The simplest way is to open a browser and enter the name that you set up in DNS, and for which we just added the ingress.</p>
  
  <p>You may also use a command line tool like curl to check the ingress.</p>
  
  <pre><code>
$ curl -kivL -H 'Host: www.example.com' 'http://203.0.113.2'
  </code></pre>
  
  <p>The options on this curl command will provide verbose output, following any redirects, show the TLS headers in the output, and not error on insecure certificates. With ingress-nginx-controller, the service will be available with a TLS certificate, but it will be using a self-signed certificate provided as a default from the ingress-nginx-controller. Browsers will show a warning that this is an invalid certificate. This is expected and normal, as we have not yet used cert-manager to get a fully trusted certificate for our site.</p>
  
  <p>Warning: It is critical to make sure that your ingress is available and responding correctly on the internet. This quick-start example uses Let's Encrypt to provide the certificates, which expects and validates both that the service is available and that during the process of issuing a certificate uses that validation as proof that the request for the domain belongs to someone with sufficient control over the domain.</p>

  <h2>Step 5 - Deploy cert-manager</h2>
  <p>We need to install cert-manager to do the work with Kubernetes to request a certificate and respond to the challenge to validate it. We can use Helm or plain Kubernetes manifests to install cert-manager.</p>
  <p>Since we installed Helm earlier, we'll assume you want to use Helm; follow the Helm guide. For other methods, read the installation documentation for cert-manager.</p>
  <p>cert-manager mainly uses two different custom Kubernetes resources - known as CRDs - to configure and control how it operates, as well as to store state. These resources are Issuers and Certificates.</p>
  <p>Issuers</p>
  <p>An Issuer defines how cert-manager will request TLS certificates. Issuers are specific to a single namespace in Kubernetes, but there's also a ClusterIssuer which is meant to be a cluster-wide version.</p>
  <p>Take care to ensure that your Issuers are created in the same namespace as the certificates you want to create. You might need to add <code>-n my-namespace</code> to your <code>kubectl create</code> commands.</p>
  <p>Your other option is to replace your Issuers with ClusterIssuers; ClusterIssuer resources apply across all Ingress resources in your cluster. If using a ClusterIssuer, remember to update the Ingress annotation <code>cert-manager.io/issuer</code> to <code>cert-manager.io/cluster-issuer</code>.</p>
  <p>If you see issues with issuers, follow the Troubleshooting Issuing ACME Certificates guide.</p>
  <p>More information on the differences between Issuers and ClusterIssuers - including when you might choose to use each can be found on Issuer concepts.</p>
  <p>Certificates</p>
  <p>Certificates resources allow you to specify the details of the certificate you want to request. They reference an issuer to define how they'll be issued.</p>
  <p>For more information, see Certificate concepts.</p>
</body>
</html>



