# alpha-cluster
initial homelab K8s cluster

### Flux Bootstrap

GENERATE PAT token in github and export it in terminal as:

```
export GITHUB_TOKEN= YOUR_PAT_TOKEN
export GITHUB_USER= YOUR_USER
```

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=alpha-cluster \
  --branch=main \
  --path=./clusters/staging \
  --personal

kubectl exec -it linkding-55d4d565f-mvnwn -- python manage.py createsuperuser --username=oliver --email=oliver@example.com

### Create secrets

brew install sops age

age-keygen -o age.agekey

export AGE_PUBLIC=

kubectl create secret generic test-secret \
--from-literal=user=admin \
--from-literal=password=oliver \
--dry-run=client \
-o yaml > test-secret.yaml


sops --age=$AGE_PUBLIC \
--encrypt --encrypted-regex '^(data|stringData)$' --in-place linkding-container-env-secret.yaml


cat age.agekey |
kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin

#### Create Secret containing user & pwd for deployment:

kubectl create secret generic linkding-container-env \
--from-literal=LD_SUPERUSER_NAME=YOUR_USER_NAME \
--from-literal=LD_SUPERUSER_PASSWORD=YOUR_PWD \
--dry-run=client \
-o yaml > linkding-container-env-secret.yaml

mv linkding-container-env-secret.yaml apps/staging/linkding