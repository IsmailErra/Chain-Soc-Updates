# ChainSOC -- updates

---

## 1. Objectif du projet

ChainSOC est un prototype de centre de securite (SOC) distribue.

L'objectif est simple : automatiser la surveillance des logs entre plusieurs machines, detecter les activites suspectes, generer des alertes, et stocker les logs de maniere securisee.

Le systeme repose sur trois machines virtuelles qui communiquent entre elles via SSH.

---

## 2. Architecture

Le projet utilise trois machines virtuelles interconnectees sur un reseau local :

| Machine   | Role                                            |
|-----------|--------------------------------------------------|
| Target    | Genere les logs et simule des activites suspectes |
| Watcher   | Surveille, detecte les menaces, genere des alertes |
| Vault     | Recoit et stocke les logs de maniere securisee    |

### Flux de fonctionnement

```
Target                    Watcher                    Vault
(genere les logs)  --->  (recupere + analyse)  --->  (stocke les logs)
                          |
                          v
                     alerts.txt
```

**En resume :**
Target genere les logs --> Watcher les recupere automatiquement --> Watcher detecte les logs suspects et cree des alertes --> Watcher envoie les logs vers Vault --> Vault stocke tout.

---

## 3. Ce que nous avons realise

### Etape 1 -- Configuration reseau

Les trois machines virtuelles ont ete configurees sur le meme reseau local avec des adresses IP statiques. La connectivite a ete testee et validee entre toutes les machines.

![Configuration reseau de la machine Target](screenshots/target_network_config.png)

![Configuration Netplan appliquee](screenshots/target_netplan_config.png)

---

### Etape 2 -- Communication SSH

SSH a ete installe et active sur toutes les machines. Un utilisateur dedie a ete cree sur Target pour permettre la connexion a distance.

Pour eviter de saisir un mot de passe a chaque connexion, nous avons mis en place une authentification par cle SSH :

```bash
ssh-keygen
ssh-copy-id target@192.168.233.103
```

Cela permet au Watcher de se connecter automatiquement a Target et a Vault sans intervention humaine.

![Service SSH actif sur Target](screenshots/target_ssh_service_setup.png)

![Connexion SSH du Watcher vers le Vault](screenshots/watcher_ssh_to_vault.png)

![Generation de la cle SSH sur le Watcher](screenshots/watcher_ssh_key_generation.png)

![Authentification par cle reussie](screenshots/watcher_ssh_key_auth_success.png)

---

### Etape 3 -- Generation de logs sur Target

Sur la machine Target, nous avons utilise la commande `logger` pour generer des logs de test dans le journal systeme :

```bash
logger "chainsoc_test_ALERT"
```

Ces logs simulent une activite suspecte que le Watcher doit detecter.

![Generation de log sur Target](screenshots/target_log_generation.png)

![Verification du log dans le journal](screenshots/target_log_verification.png)

---

### Etape 4 -- Recuperation automatique des logs

Un script Bash (`check_logs.sh`) a ete cree sur le Watcher. Ce script se connecte a Target via SSH et recupere les logs recents contenant le mot cle `chainsoc_test`.

![Script check_logs.sh](screenshots/watcher_check_logs_script.png)

![Logs recuperes depuis Target](screenshots/watcher_log_retrieval_result.png)

---

### Etape 5 -- Envoi automatique vers Vault

Un second script (`send_to_vault.sh`) a ete cree. Il fait tout en une seule execution :

1. Se connecte a Target et recupere les logs
2. Verifie si un log suspect est present
3. Si oui, genere une alerte
4. Envoie le log vers la machine Vault

![Script send_to_vault.sh](screenshots/watcher_alert_detection_script.png)

![Execution du script et envoi vers Vault](screenshots/watcher_send_to_vault_execution.png)

---

### Etape 6 -- Generation d'alertes

Quand le script detecte un log suspect, il affiche un message d'alerte et enregistre le log dans `alerts.txt` sur le Watcher :

```
ALERT: Suspicious log detected!
```

![Alerte detectee et enregistree](screenshots/watcher_alert_detection_result.png)

---

### Etape 7 -- Automatisation avec cron

Pour que le systeme fonctionne en continu sans intervention, le script `send_to_vault.sh` a ete ajoute au planificateur de taches `cron` :

```bash
crontab -e
* * * * * /home/hajar/send_to_vault.sh
```

Chaque minute, le systeme execute automatiquement le cycle complet :
- Recuperation des logs depuis Target
- Detection des activites suspectes
- Creation d'alertes
- Envoi des logs vers Vault
- Stockage des logs

![Configuration du cron](screenshots/cron_automation_setup.png)

![Logs accumules sur Vault apres plusieurs cycles](screenshots/vault_cron_logs_accumulated.png)

---

## 4. Scripts crees

| Script              | Fonction                                                    |
|---------------------|-------------------------------------------------------------|
| `check_logs.sh`     | Recupere les logs recents depuis Target via SSH              |
| `send_to_vault.sh`  | Detecte les logs suspects, genere des alertes, envoie vers Vault |

---

## 5. Test du systeme

Pour prouver que l'automatisation fonctionne de bout en bout, nous avons realise le test suivant :

**1. Generer un log suspect sur Target :**

```bash
logger "chainsoc_test_FINAL"
```

![Log de test genere sur Target](screenshots/target_suspicious_log_generation.png)

**2. Attendre une minute** (le temps que le cron s'execute).

**3. Verifier les alertes sur Watcher :**

```bash
cat alerts.txt
```

Le fichier contient bien le log suspect detecte automatiquement.

![Alertes enregistrees](screenshots/watcher_alert_detection_result.png)

**4. Verifier le stockage sur Vault :**

```bash
cat vault_logs.txt
```

Le log a bien ete recu et stocke sur la machine Vault.

![Logs stockes sur Vault](screenshots/vault_log_storage_verification.png)

Ce test prouve que le systeme complet fonctionne de maniere autonome : detection, alerte et stockage, sans aucune intervention manuelle.

---

## 6. Resultat final

Le pipeline automatise fonctionne comme suit :

```
Target genere un log
      |
      v
Watcher recupere le log automatiquement
      |
      v
Watcher detecte l'activite suspecte
      |
      v
Watcher genere une alerte (alerts.txt)
      |
      v
Watcher envoie le log vers Vault
      |
      v
Vault stocke le log (vault_logs.txt)
```

Tout ce cycle s'execute automatiquement chaque minute grace au cron.

---

## 7. Limites actuelles

Les composants suivants sont des implementations temporaires utilisees pour valider rapidement le fonctionnement du systeme :

| Composant         | Statut                               |
|-------------------|--------------------------------------|
| `alerts.txt`      | Stockage temporaire des alertes       |
| `vault_logs.txt`  | Stockage temporaire des logs sur Vault |

Ces fichiers texte ont permis de tester et de prouver le bon fonctionnement de l'automatisation. Ils seront remplaces par des solutions plus robustes dans les versions futures.

---

## 8. Ameliorations futures

- **IPFS** -- Stockage decentralise des logs pour garantir leur disponibilite
- **Blockchain** -- Enregistrement des logs sur une blockchain pour garantir leur integrite et immutabilite
- **Smart contracts** -- Verification automatique des logs via des contrats intelligents
- **Dashboard** -- Interface de visualisation en temps reel des alertes et des logs

---

## Technologies utilisees

| Technologie         | Utilisation                                  |
|---------------------|----------------------------------------------|
| VMware Workstation  | Virtualisation des trois machines            |
| Ubuntu (LTS)        | Systeme d'exploitation des machines virtuelles |
| SSH (OpenSSH)       | Communication securisee entre les machines    |
| Bash                | Scripts d'automatisation                     |
| Cron                | Planification de l'execution automatique      |
| journalctl / syslog | Generation et consultation des logs           |

---

## Conclusion

ChainSOC demontre un prototype fonctionnel de SOC distribue. Le systeme collecte automatiquement les logs, detecte les activites suspectes, genere des alertes et stocke les donnees de maniere securisee, le tout sans aucune intervention manuelle. L'automatisation complete a ete validee par des tests concrets.
