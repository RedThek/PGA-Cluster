# PGA-Cluster

Server side clustering for high data availability.

# Rôles Ansible — haproxy, galera, glusterfs

À copier dans `ansible/roles/` de ton dépôt PGA-Cluster (le rôle `keepalived` généré précédemment reste inchangé, à côté de ceux-ci).

## Prérequis avant la première exécution

Sur la machine de contrôle (node1) :

```bash
apt install -y python3-pymysql
ansible-galaxy collection install community.mysql
```

(`community.mysql` fournit le module `mysql_user` utilisé par le rôle `galera` — le paquet `ansible` complet l'inclut généralement déjà, mais mieux vaut le confirmer explicitement.)

## Secrets

Créer le vault chiffré à partir de l'exemple fourni :

```bash
ansible-vault create group_vars/vault.yml
# coller le contenu de group_vars/vault.yml.example en y mettant tes vraies valeurs
rm group_vars/vault.yml.example   # ne garder que la version chiffrée dans le repo
```

## Premier bootstrap du cluster Galera (une seule fois, node1 uniquement)

```bash
ansible-playbook site.yml --limit node1 -e "galera_first_run=true" --ask-vault-pass
```

Ne jamais repasser `galera_first_run=true` après ce premier lancement, sous peine de re-bootstrap (split-brain) sur un cluster déjà existant.

## Exécutions normales (rejoindre / maintenir le cluster)

```bash
ansible-playbook site.yml --ask-vault-pass
```

## Finaliser GlusterFS (une fois les 3 nœuds déployés et joignables)

```bash
ansible-playbook site.yml --tags glusterfs -e "glusterfs_finalize=true" --ask-vault-pass
```

(ou directement dans `site.yml` si tu préfères ne pas gérer de tags — voir le rôle `glusterfs`, la variable par défaut est `false` pour ne rien casser tant que node2/node3 n'existent pas.)

## Vérification à blanc avant toute exécution réelle

```bash
ansible-playbook site.yml --limit node1 --check --diff --ask-vault-pass
```
