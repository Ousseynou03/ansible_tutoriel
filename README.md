### Ceci sont les étapes pour configuer efficacement Ansible

## NB : Pour commencer, tu peux juste créer deux VMs avec VMWare et le tour est joué pour tester tes premiers pas avec Ansible

## Dans un premier temps il faut se rassurer que openssh est dèjà dans les serveurs cible

si ce n'est pas le cas, effectuez l'installation avec : sudo apt install openssh-server (Sur ubuntu)

## Effectuez la connexion ssh une premiére fois pour chaque serveur et répondre par yes pour enregistrer le serveur

ssh username@Ip ensuite rentrer le password demandé

## Créez une paire de clé ssh pour le user de la machine où les commandes ansibles seront exécutées
 # Vérifie d'abord si dés clé pub/pri existent : ls -ls ~/.ssh
 Sinon il faut en générer une : 
 ssh-keygen -t ed_ed25519 -C "just comment"


## Copy cette clé dans chaque serveur

Effectuez cela avec la commande : ssh-copy-id -i ~/.ssh/id_ed25519.pub user@IP

dans le serveur , vérifie que la clé a été bien copié au niveau de authorized_keys avec la commande : cat ~/.ssh/authorized_keys

## Créez une clé ssh qui est spécifique à Ansible et copiez cette derniére aussi dans chaque serveur 


ssh-keygen -t ed25519 -C "ansible"
Generating public/private ed25519 key pair.
donner le chemin de votre avec le ansible (Ex. ~/home/.ssh/ansible) sinon cette nouvelle clé va écraser l'ancienne.

NB : Puisque une clé sous le non de id_ed25519 existe déjà, pour ne pas l'écraser, on crée un nouveau nom pour clé ansible

## Nous allons à présent commencer à interagir avec Ansible.

On doit d'abord l'installer dans la machine hôte avec la commande suivante : <code> pip install ansible </code>

## NB : Ansible n'est pas compatible avec windows, veuillez utiliser un wsl et exécuter là-bas toutes les commandes relatif à Ansible.
## Vérifiez que ansible peut se connecter aux machines via ssh en lançant avec le module ping

ansible all --key-file ~/.ssh/ansible -i inventory.ini -m ping

Le résultat donne : 
<code>
$ ansible all --key-file ~/.ssh/ansible -i inventory.ini -m ping
192.168.200.130 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.200.131 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
</code>


## ansible all --key-file ~/.ssh/ansible -i inventory.ini -m gather_facts

Cette commande nous récupérer des macros informations sur les différents que l'on a spécifié dans le inventory.ini

## Commandes qui nécessitent sudo :

Dans certaines des opérations sur Linux, le sudo est obligatoire de ce fait, il faut savoir le spécifier à Ansible

Par exemple cette commande suivante va échouer :

    <code> $ ansible all --key-file ~/.ssh/ansible -i inventory.ini -m apt -a update_cache=true </code>

    Il faut spécifier comment ansible doit utiliser le sudo avec la nouvelle commande : 
    <code> $ ansible all --key-file ~/.ssh/ansible -i inventory.ini -m apt -a update_cache=true --become --ask-become-pass </code>

    NB : Cette commande est utilisé si et seulement tous les serveurs ont le même password , sinon il faut donner le password en même temps dans le inventory comme ceci : 

    <code> 
            [all]
        192.168.200.130 ansible_user=srver1 ansible_become_password=srver1
        192.168.200.131 ansible_user=srver2 ansible_become_password=srver2

    </code>

    Ainsi on laisse juste le --become pour que ansible puisse effectuer des commande avec sudo :

    <code> $ ansible all --key-file ~/.ssh/ansible -i inventory.ini -m apt -a update_cache=true --become </code>

## Installation d'un paquet dans tous les serveurs (vim-nox)
    <code> $ ansible all --key-file ~/.ssh/ansible -i inventory.ini -m apt -a name=vim-nox --become </code>

On voit donc qu'avec ces commandes, on peut installer tout ce que l'on veut dans tous les serveurs, mais cela risque de ne pas être pratique du tout s'il y'a plusieurs installations à effectuer.

## On va de ce pas, créer ce qu'on appelle des playbooks pour centraliser toutes les installations à effectuer dans un script bien organisé.

Aprés avoir écris le script d'installation, exécuter la commande suivante pour installer apapche2 sur les deux machines :
voici le playbook d'installation d'apache


    <code> 
    ---
    - hosts: all
        become: true
        tasks:

        - name: Install Apache2 package
            apt:
            name: apache2
    </code>

    <code> ansible-playbook --key-file ~/.ssh/ansible -i inventory.ini install_apache.yml </code>


