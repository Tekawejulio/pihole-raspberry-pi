# Pi-hole sur Raspberry Pi (Docker)

Déploiement de [Pi-hole](https://pi-hole.net/) sur un Raspberry Pi via Docker Compose, pour bloquer les publicités et trackers au niveau DNS sur le réseau local.

## Stack

- **Matériel** : Raspberry Pi (Debian/Raspberry Pi OS, Trixie, arm64)
- **Conteneurisation** : Docker CE + Docker Compose
- **Application** : Pi-hole (image officielle `pihole/pihole:latest`)

## Prérequis

- Raspberry Pi avec Raspberry Pi OS (ou Debian) installé
- Accès SSH
- IP fixe recommandée pour le Pi

## Installation de Docker

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### ⚠️ Problème rencontré : `iptables` introuvable

**Symptôme** : `apt install iptables` renvoie *"has no installation candidate"*.

**Cause** : index APT local corrompu (souvent dû à un `apt update` interrompu).

**Solution** :
```bash
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

## Déploiement de Pi-hole

Fichier `docker-compose.yml` :

```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    network_mode: "host"
    environment:
      TZ: 'Europe/Brussels'
      FTLCONF_webserver_api_password: 'CHANGE_MOI'
      FTLCONF_dns_listeningMode: 'all'
    volumes:
      - './etc-pihole:/etc/pihole'
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

Lancement :
```bash
sudo docker compose up -d
```

### ⚠️ Problème rencontré : blocage DNS invisible depuis le réseau local

**Symptôme** : Pi-hole fonctionnait en local sur le Pi, mais toute requête DNS depuis un autre appareil du réseau expirait en timeout.

**Diagnostic** : `tcpdump` a montré que les requêtes arrivaient bien au conteneur, mais qu'aucune réponse ne repartait. Un test direct avec `dig` sur l'IP interne du conteneur a confirmé que Pi-hole répondait correctement — le problème venait du trajet retour UDP.

**Cause** : en mode réseau `bridge` (par défaut), le NAT géré par Docker perdait le chemin de retour des réponses DNS en UDP.

**Solution** : passage en `network_mode: "host"`, qui supprime la couche NAT intermédiaire.

**Vérification finale** :
nslookup doubleclick.net 192.168.129.200
Adresses: 0.0.0.0

nslookup google.com 192.168.129.200
Adresses: 172.217.171.110


## Compétences mises en œuvre

- Administration Linux (Debian/Raspberry Pi OS)
- Docker et Docker Compose
- Réseaux : DNS, DHCP, NAT, TCP/UDP
- Diagnostic réseau : `tcpdump`, `dig`, `nslookup`, `ss`
- Résolution de problèmes en autonomie
- Documentation technique

## Auteur

Julio — [GitHub](https://github.com/Tekawejulio)
