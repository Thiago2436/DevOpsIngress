


<!DOCTYPE html>
<html>
<head>
  <title>Securing NGINX-ingress</title>
</head>
<body>
  <h1>Securing NGINX-ingress</h1>
  <p>This tutorial will detail how to install and secure ingress to your cluster using NGINX.</p>
  <ol>
    <li>Step 1 - Install Helm</li>
    <p>Skip this section if you have helm installed.</p>
    <p>To install Helm, follow the instructions below:</p>
    <ol>
      <li>Open your terminal.</li>
      <li>Run the following command to download and install Helm:</li>
    </ol>
    <pre><code>$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash</code></pre>
    <p>Once Helm is installed, you can verify the installation by running the following command:</p>
    <pre><code>$ helm version</code></pre>
    <li>Step 2 - Add NGINX Helm repository</li>
    <p>To add the NGINX Helm repository, execute the following command:</p>
    <pre><code>$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx</code></pre>
    <p>After adding the repository, you can update your local Helm chart repository cache:</p>
    <pre><code>$ helm repo update</code></pre>
    <li>Step 3 - Install NGINX-ingress</li>
    <p>Use the following command to install NGINX-ingress:</p>
    <pre><code>$ helm install nginx-ingress ingress-nginx/ingress-nginx</code></pre>
    <p>This command will install the NGINX-ingress controller in your Kubernetes cluster.</p>
    <li>Step 4 - Secure NGINX-ingress</li>
    <p>To secure the NGINX-ingress controller, follow these steps:</p>
    <ol>
      <li>Open the values.yaml file of the NGINX-ingress release.</li>
      <li>Find the <code>controller</code> section.</li>
      <li>Set the <code>enabled</code> parameter to <code>true</code> to enable SSL.</li>
      <li>Specify the <code>tlsSecret</code> parameter with the name of your TLS secret.</li>
      <li>Save the file and exit the editor.</li>
      <li>After making these changes, you can upgrade the NGINX-ingress release to apply the new configuration:</li>
    </ol>
    <pre><code>$ helm upgrade nginx-ingress ingress-nginx/ingress-nginx -f values.yaml</code></pre>
    <p>Once the upgrade is complete, your NGINX-ingress controller will be secured with SSL.</p>
    <li>Step 5 - Configure Ingress rules</li>
    <p>Open your preferred text editor and create an ingress.yaml file.</p>
    <p>Add the necessary ingress rules to the file.</p>
    <p>Save the file.</p>
    <p>Apply the ingress rules by running the following command:</p>
    <pre><code>$ kubectl apply -f ingress.yaml</code></pre>
    <li>Step 6 - Verify the configuration</li>
    <p>Use the following command to check the status of your ingress:</p

>
    <pre><code>$ kubectl get ingress</code></pre>
    <p>The output will show the status of your ingress along with other details.</p>
    <li>Step 7 - Test the ingress</li>
    <p>Test the ingress configuration by accessing the specified hostname or URL in your web browser.</p>
  </ol>
</body>
</html>


