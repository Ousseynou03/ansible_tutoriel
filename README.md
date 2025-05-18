### Ceci sont les étapes pour configuer efficacement Ansible

## Dans un premier temps il faut se rassurer que openssh est dèjà dans les serveurs cible

si ce n'est pas le cas, effectuez l'installation avec : sudo apt install openssh-server (Sur ubuntu)

## Effectuez la connexion ssh une premiére fois pour chaque serveur et répondre par yes pour enregistrer le serveur

ssh username@Ip ensuite rentrer le password demandé

## Créez une paire de clé ssh pour le user de la machine où les commandes ansibles seront exécutées
 # Vérifie d'abord si dés clé pub/pri existent : ls -ls ~/.ssh
 Sinon il faut en générer une : 
 ssh-keygen -t ed_ed25519 -c "just comment"


## Copy cette clé dans chaque serveur

Effectuez cela avec la commande : ssh-copy-id -i ~/.ssh/id_ed25519.pub user@IP

dans le serveur , vérifie que la clé a été bien copié au niveau de authorized_keys avec la commande : cat ~/.ssh/authorized_keys

## Créez une clé ssh qui est spécifique à Ansible et copiez cette derniére aussi dans chaque serveur 

$ ssh-keygen -t ed25519 -C "ansible"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/c/Users/weuzi/.ssh/id_ed25519): /c/Users/weuzi/.ssh/ansible

NB : Puisque une clé sous le non de id_ed25519 existe déjà, pour ne pas l'écraser, on crée un nouveau nom pour clé ansible

