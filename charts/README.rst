Helm Charts for Deploying RENGA on Kubernetes
=============================================

Testing locally
---------------

Requires minikube, kubectl and helm.

.. code-block:: console

    $ minikube start
    $ helm init
    $ helm repo add renga https://swissdatasciencecenter.github.io/helm-charts/
    $ helm dep build renga
    $ helm install --name nginx-ingress --namespace kube-system stable/nginx-ingress --set controller.hostNetwork=true
    $ helm install --name renga --namespace renga \
        -f minikube-values.yaml \
        --set global.renga.domain=$(minikube ip) \
        --set ui.gitlabUrl=$(minikube ip)/gitlab \
        renga

The platform takes some time to start, to check the pods status do:

.. code-block:: console

    $ kubectl -n renga get po --watch

and wait until all pods are running.
Now, we can go to: :code:`http://$(minikube-ip)/`

Deploying from a Helm repository
--------------------------------

.. code-block:: console

    $ minikube start
    $ helm init
    $ helm repo add renga https://swissdatasciencecenter.github.io/helm-charts/
    $ helm fetch --devel renga/renga
    $ ls renga-*.tgz
    renga-0.1.0-XXXXXX.tgz
    $ helm install --name renga --namespace renga \
        -f minikube-values.yaml \
        --set global.renga.domain=$(minikube ip) \
        --set ui.gitlabUrl=$(minikube ip)/gitlab \
        renga-0.1.0-XXXXXX.tgz
