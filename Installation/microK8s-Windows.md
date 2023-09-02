![image](https://github.com/tahauser/k8s-exercices/assets/90762839/d02027bb-b4b4-47c8-b486-f9760306cfd7)MicroK8s sous Windows

### Quésaco ?
MicroK8s est une option pour déployer un cluster Kubernetes à un seul nœud en tant que package unique pour cibler des stations de travail et des appareils IoT (Internet of Things).

Canonical, le créateur de Linux Ubuntu, en est à l’origine et fait la maintenance de MicroK8s.

Vous pouvez installer MicroK8s sur Linux, Windows et MacOS.

### Installer sur Windows
Pour exécuter MicroK8s sur Windows, utilisez Multipass. 
Multipass est un gestionnaire de machines virtuelles léger pour Linux, Windows et MacOS.
Télécharger et installer Multipass https://multipass.run
```
multipass launch --name microk8s-vm --memory 1G --disk 4G
```
```
multipass list
```

Une fois que vous avez reçu la confirmation du lancement pour microk8s-vm, exécutez la commande suivante pour accéder à cette instance :
```
multipass shell microk8s-vm
```

Installez l’application Snap MicroK8s
```
sudo snap install microk8s --classic
```

Pour vérifier l’état de l’installation, exécutez la commande
```
sudo microk8s.status --wait-ready
```

### Activer les modules complémentaires DNS, Tableau de bord et Registre
```
sudo microk8s.enable dns dashboard registry
```

DNS : Déploie le service coreDNS.
Tableau de bord : Déploie le service kubernetes-dashboard et plusieurs autres services qui prennent en charge cette fonctionnalité. Il s’agit d’une interface utilisateur web à usage général pour les clusters Kubernetes.
Registre : Déploie un registre privé et plusieurs services qui prennent en charge cette fonctionnalité. Pour stocker des conteneurs privés, utilisez ce registre.


