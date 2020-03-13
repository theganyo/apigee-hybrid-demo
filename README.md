Demo Hybrid

1. Install Apigee hybrid

2. Ensure policy is not disabled

    kubectl -n istio-system get cm istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks

Response: `disablePolicyChecks: false`

3. Enable injection

    kubectl label namespace default istio-injection=enabled

4. Deploy httpbin and test ok

    export GATEWAY_URL=???

    kubectl apply -f httpbin.yaml
    kubectl apply -f httpbin-gateway.yaml

    curl http://${GATEWAY_URL}/headers

5. (Optional) Check that Istio policy is working using simple denier

    kubectl apply -f denier.yaml

Let Istio catch up then try again

    curl http://${GATEWAY_URL}/headers

Response: `PERMISSION_DENIED:denier.default:Not allowed`

Delete denier

    kubectl delete -f denier.yaml

Verify ok again

    curl http://${GATEWAY_URL}/headers

6. Provision hybrid for Istio adapter

    export ORG=???
    export ENV=???
    export EMAIL=???
    
    export TOKEN=$(gcloud auth print-access-token)
    echo $TOKEN
    
    apigee-istio-linux provision \
      --org $ORG \
      --env $ENV \
      --hybrid \
      --developer-email $EMAIL \
      --routerBase $HYBRID_RUNTIME_HOST \
      --token $TOKEN > handler.yaml

(use `apigee-istio-darwin` for mac)

7. Deploy adapter

    kubectl apply -f adapter.yaml
    
Ensure it's running

    kubectl -n apigee get po -l app=apigee-adapter

8. Apply Istio definitions, handler, rule, test

    kubectl apply -f definitions.yaml
    kubectl apply -f handler.yaml
    kubectl apply -f rule.yaml
       
9. Follow standard Apigee adapter directions from this point:

    https://github.com/apigee/istio-mixer-adapter#authentication-test

