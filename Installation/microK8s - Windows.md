MicroK8s sous Windows

### Quésaco ?
MicroK8s est une option pour déployer un cluster Kubernetes à un seul nœud en tant que package unique pour cibler des stations de travail et des appareils IoT (Internet of Things).

Canonical, le créateur de Linux Ubuntu, en est à l’origine et fait la maintenance de MicroK8s.

Vous pouvez installer MicroK8s sur Linux, Windows et MacOS.

### Installer sur Windows
Pour exécuter MicroK8s sur Windows, utilisez Multipass. 
Multipass est un gestionnaire de machines virtuelles léger pour Linux, Windows et MacOS.
Télécharger et installer Multipass https://multipass.run
```
multipass launch --name microk8s-vm --mem 1G --disk 4G
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
