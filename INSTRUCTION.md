Create the kind cluster from the provided config:
kind create cluster --name todoapp --config cluster.yml
Bootstrap the cluster (installs NGINX ingress controller and deploys the app):
chmod +x bootstrap.sh && ./bootstrap.sh
Apply the Ingress manifest (if not already applied by bootstrap):
kubectl apply -f ./.infrastructure/ingress/ingress.yml

kubectl wait --for=condition=ready pod/$POD -n todoapp
kubectl get pod $POD -n todoapp -o jsonpath='{.spec.serviceAccountName}'

# Get the pod name
POD_NAME=$(kubectl get pods -n todoapp -l app=todoapp -o jsonpath='{.items[0].metadata.name}')

# Exec into the pod and run the correct curl command
kubectl exec -it $POD_NAME -n todoapp -- /bin/sh -c '
  TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) && \
  CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt && \
  curl --cacert $CACERT --header "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets