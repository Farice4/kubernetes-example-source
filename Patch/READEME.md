### Create deployment

"""
kubectl create -f test-deployment.yaml
"""

### Patch deployment
"""
kubectl patch deployment patch-demon --patch "$(cat patch-file.yaml)"
"""