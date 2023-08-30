Dans cet exercice vous allez activer la génération des logs d'audit sur un cluster créé avec kubeadm

## Prérequis

Assurez-vous d'avoir accès à un cluster créé avec *kubeadm*.

Si vous souhaitez créer un cluster rapidement, vous pouvez installer [Multipass](https://multipass.run) et lancer la commande suivante, celle-ci crée des VMs Ubuntu et installe Kubernetes avec Kubeadm.

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

## A propos des logs d'audit

Le niveau de logging des logs d'audit peut être défini à différents niveau:

- None: les évènements associés ne sont pas loggués
- Metadata: les meta-données des évènements sont loggées (requestor, timestamp, resource, verb, etc.) mais les body des requêtes et réponses ne le sont pas
- Request: les meta-données et le body de la requète sont loggés, le body de la réponse n'est par contre pas loggué
- RequestResponse: les méta-données et les body des requêtes et réponses sont loggués

Dans le cadre de cet exercice nous allons considérer une Policy d'audit qui loggue les meta-données de chaque requète envoyée à l'API Server. Nous utiliserons pour cela le fichier de configuration suivant:

```
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

## A vous de jouer

En utilisant la documentation suivante: [https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/) configurez l'API Server afin d'activer les logs d'audit. Assurez-vous de configurer un *log backend*.

Créez ensuite un pod simple:

```
kubectl run nginx --image=nginx:1.20
```

Vérifiez que la création de ce Pod est bien logguée dans le fichier contenant les logs d'audit.