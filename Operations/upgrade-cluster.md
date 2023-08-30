Dans cet exercice, nous allons mettre à jour un cluster Kubernetes depuis la version 1.24.6 vers la version 1.25.2

## Prérequis

Avoir accès à un cluster dans la version 1.24.6 créé avec *kubeadm*.

Si vous souhaitez créer un cluster local 1.24.6 rapidement, vous pouvez lancer la commande suivante. Celle-ci crée des VMs Ubuntu en utilisant [Multipass](https://multipass.run) et installent Kubernetes avec Kubeadm*.

```
curl https://luc.run/k8s.sh | bash -s -- -v 1.24.6 -w 2
```

En quelques minutes vous aurez un cluster dans la version souhaitée, celui-ci étant constitué d'un node control-plane et de 2 nodes worker (worker1 et worker2).

Depuis votre machine locale vous pourrez alors configurer kubectl:

```
export KUBECONFIG=$PWD/kubeconfig.cfg
```

puis lister les nodes du cluster:

```
$ kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
control-plane   Ready    control-plane   92s   v1.24.6
worker1         Ready    <none>          71s   v1.24.6
worker2         Ready    <none>          51s   v1.24.6
```

## Mise à jour du node control-plane

La mise à jour commence par les nodes du control-plane (un seul dans le cadre de cet exercice)

### Mise à jour de kubeadm

Depuis un shell sur le node control-plane, lancez la commande suivante afin de lister les versions de *kubeadm* actuellement disponibles:

```
ubuntu@control-plane:~# sudo apt update && sudo apt-cache policy kubeadm
kubeadm:
  Installed: 1.24.6-00
  Candidate: 1.25.2-00
  Version table:
     1.25.2-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
     1.25.1-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
     1.25.0-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
 *** 1.24.6-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
        100 /var/lib/dpkg/status
     1.24.5-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
     1.24.4-00 500
        500 http://apt.kubernetes.io kubernetes-xenial/main arm64 Packages
     1.24.3-00 500
...
```

Note: il est possible que vous obteniez un résultat différent, les mises à jour de Kubernetes étant relativement fréquentes (3 releases mineures par an depuis la version 1.22.0). Nous effectuerons ici une mise à jour vers la version 1.25.2.

Utilisez les commandes suivantes afin de mettre à jour *kubeadm* sur le node control-plane:

```
VERSION=1.25.2-00

sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=$VERSION
sudo apt-mark hold kubeadm
```

### Passage du node en mode Drain

En utilisant la commande suivante depuis votre machine locale, passez le node control-plane en *drain* de façon à ce que les Pods applicatifs (si il y en a) soient re-schédulés sur les autres nodes du cluster.

```
$ kubectl drain control-plane --ignore-daemonsets
node/control-plane cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/cilium-fzlnz, kube-system/kube-proxy-s96sb
evicting pod kube-system/coredns-6d4b75cb6d-slc47
evicting pod kube-system/cilium-operator-75d8cffd95-f9g6f
evicting pod kube-system/coredns-6d4b75cb6d-nfxz8
pod/cilium-operator-75d8cffd95-f9g6f evicted
pod/coredns-6d4b75cb6d-nfxz8 evicted
pod/coredns-6d4b75cb6d-slc47 evicted
node/control-plane drained
```

### Upgrade du node

Depuis un shell root sur le node control-plane, vous pouvez à présent lancer la simulation de la mise à jour avec la commande suivante:

```
ubuntu@control-plane:~# sudo kubeadm upgrade plan
...
You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.25.2
...
```

Si vous avez un résultat similaire à celui ci-dessus, c'est que la simulation s'est déroulée correctement. Vous pouvez alors lancer la mise à jour:

```
ubuntu@control-plane:~# sudo kubeadm upgrade apply v1.25.2
```

Après quelques minutes vous devriez obtenir le message suivant:

```
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.25.2". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

### Passage du node en mode Uncordon

Modifiez le node control-plane de façon à le rendre de nouveau "schedulable".

```
$ kubectl uncordon control-plane
node/control-plane uncordoned
```

Note: dans le cas d'un cluster avec plusieurs node control-plane, il faudrait également mettre à jour kubeadm sur les autres node puis lancer la commande suivante sur chacun d'entres eux:

```
$ kubeadm upgrade NODE_IDENTIFIER
```

### Mise à jour de kubelet et kubectl

Depuis un shell sur le node control-plane, utilisez la commande suivante afin de mettre à jour *kubelet* et *kubectl*:

```
VERSION=1.25.2-00

sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION
sudo apt-mark hold kubelet kubectl
```

Redémarrez ensuite *kulelet*

```
sudo systemctl restart kubelet
```

Vérifiez que le node control-plane est à présent dans la nouvelle version:

```
NAME            STATUS   ROLES           AGE     VERSION
control-plane   Ready    control-plane   7m53s   v1.25.2
worker1         Ready    <none>          7m6s    v1.24.6
worker2         Ready    <none>          6m52s   v1.24.6
```

## Mise à jour des nodes workers

Effectuez les actions suivantes sur chacun des nodes worker. Les instructions suivantes détaillent les actions à effectuer sur le premier worker, il faudra ensuite faire la même chose sur le second.

### Mise à jour de kubeadm

Depuis un shell sur le worker1, lancez la commande suivante afin d'installer la version 1.25.2 du binaire *kubeadm*:

```
VERSION=1.25.2-00 

sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=$VERSION
sudo apt-mark hold kubeadm
```

### Passage du node en mode Drain

Depuis la machine locale, préparez le node pour le maintenance en la passant en mode *drain*, les Pods tournant sur le node seront reschédulés sur les autres nodes du cluster.

```
kubectl drain worker1 --ignore-daemonsets
```

### Mise à jour de la configuration de kubelet

lancez la commande suivante afin de mettre à jour la configuration de *kubelet*.

```
sudo kubeadm upgrade node
```

Vous obtiendrez un résultat similaire à celui ci-dessous:

```
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```

### Mise à jour de kubelet

Mettez ensuite à jour les binaires *kubelet* et *kubectl* à jour (toujours depuis un shell sur le node worker1):

```
VERSION=1.25.2-00

sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION
sudo apt-mark hold kubelet kubectl
```

Puis redémarrez *kubelet* à l'aide de la commande suivante:

```
sudo systemctl restart kubelet
```

Vous pouvez ensuite rendre le node "schedulable":

```
kubectl uncordon worker1
```

Vérifiez alors que le premier worker est maintenant à jour.

```
$ kubectl get node
NAME            STATUS   ROLES           AGE   VERSION
control-plane   Ready    control-plane   13m   v1.25.2
worker1         Ready    <none>          13m   v1.25.2
worker2         Ready    <none>          12m   v1.24.6
```

Effectuez à présent ces actions sur le second worker.

## Test

Votre cluster doit maintenant avoir 3 nodes dans la version 1.25.2:

```
$ kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
control-plane   Ready    control-plane   16m   v1.25.2
worker1         Ready    <none>          15m   v1.25.2
worker2         Ready    <none>          15m   v1.25.2
```
