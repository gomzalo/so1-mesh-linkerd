# :::::::::::::::::::::      PARTE 1        :::::::::::::::::::::

kubectl -n linkerd-demo port-forward svc/apache 8888:80

## Abrir Linkerd Dashboard, localmente

linkerd viz dashboard

# PARTE 1

# Inject Linkerd to demo

kubectl get -n linkerd-demo deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -

kubectl -n linkerd-demo get deployments

## Observabilidad

linkerd -n linkerd-demo stat deploy
linkerd -n linkerd-demo top deploy
linkerd -n linkerd-demo tap deploy/client

# :::::::::::::::::::::      PARTE 2        :::::::::::::::::::::

## Instalar Flagger para hacer Canary Deployments

kubectl apply -k github.com/weaveworks/flagger/kustomize/linkerd
kubectl -n linkerd rollout status deploy/flagger
kubectl create ns test && \
ubectl apply -f https://run.linkerd.io/flagger.yml
kubectl -n test rollout status deploy podinfo  
kubectl -n test port-forward svc/podinfo 9898

------- Hacer Canary deploy sin iniciarlo   ------


## Iniciar Canary

## Abrir namespace test=>traffic-split=>podinfo-primary

kubectl -n test get ev --watch

## Empezar rollout

kubectl -n test set image deployment/podinfo \
  podinfod=quay.io/stefanprodan/podinfo:1.7.1


## Desinstalar demo

kubectl delete -k github.com/weaveworks/flagger/kustomize/linkerd && \
  kubectl delete ns test
