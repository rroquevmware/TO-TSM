# Enabling Distributing Tracing in TSM and Integrating with Tanzu Observability
---
This guide documents the steps required to enable distributed tracing in TSM and to forward those traces to Tanzu Obervability  


### Configuration Steps
---
1. Create a namespace on your Kubernetes cluster called `wavefront` and make sure that this namespace is excluded from any TSM sidecar injection. Once the namespace is created deploy the wavefront proxy using helm

    Add the helm repo
    ```shell
    $ helm repo add wavefront https://wavefronthq.github.io/helm/
    ```
    Create a namespace called `wavefront` and deploy the wavefront helm chart to the `wavefront` namespace 
    ```shell
    $ helm install wavefront wavefront/wavefront \          
    --set wavefront.url=<wavefront url ie. https://vmware.wavefront.com> \    
    --set wavefront.token=<api token> \  
    --set clusterName="<kubernetes cluster name>" \
    --set proxy.zipkinPort=9411 \
    --set proxytraceZipkinApplicationName="<application name>" -n wavefront
    ```

2. Enable distrubed tracing in TSM by editing the `istio` configmap in `istio-system` namespace and setting the paramter `enableTracing: true` 
    ```shell
    $ kubectl edit cm istio -n istio-system
    ```
    Change `enableTracing:` to `true` and save
    ```yaml
    ...
    enableTracing: true 
    ...
    ```

3. For demos change the sampling rate to 100% by modifying the `istiod` deployment in `istio-system` 
    ```shell
    $ kubectl edit deployment istiod -n istio-system
    ```
    Search for `PILOT_TRACE_SAMPLING` and change `value:` to `"100"`
    ```yaml
    - name: PILOT_TRACE_SAMPLING
      value: "100"
    ```

4.  Create the following service in `istio-system` namespace
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: zipkin
      namespace: istio-system
    spec:
      type: ExternalName
      externalName: wavefront-proxy.wavefront.svc.cluster.local    

    ```
    Save the above yaml to a file ie. wv-externalname.yaml and apply it as follows
    ```shell
    $ kubectl apply -f wv-externalname.yaml -n istio-system
    ```
    At this point all traces will now be sent from TSM to Tanzu Observability!

