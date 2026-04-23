# ChainSOC -- Prototype de SOC Distribue

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

```
Target                    Watcher                    Vault
(genere les logs)  --->  (recupere + analyse)  --->  (stocke les logs)
                          |
                          v
                     alerts.txt
```

---

## 3. Ce que nous avons realise

### Etape 1 -- Configuration reseau

Les trois machines virtuelles ont ete configurees sur le meme reseau local avec des adresses IP statiques. La connectivite entre toutes les machines a ete testee et validee.

### Etape 2 -- Communication SSH

SSH a ete installe et active sur toutes les machines. Pour eviter de saisir un mot de passe a chaque connexion, nous avons mis en place une authentification par cle SSH avec `ssh-keygen` et `ssh-copy-id`. Cela permet au Watcher de se connecter automatiquement a Target et a Vault sans intervention humaine.

![Authentification SSH par cle reussie entre les machines](screenshots/watcher_ssh_key_auth_success.png)

### Etape 3 -- Generation de logs sur Target

Sur la machine Target, nous utilisons la commande `logger` pour generer des logs de test dans le journal systeme. Ces logs simulent une activite suspecte que le Watcher doit detecter.

```bash
logger "chainsoc_test_ALERT"
```

### Etape 4 -- Recuperation automatique des logs

Le script `check_logs.sh` sur le Watcher se connecte a Target via SSH et recupere les logs recents contenant le mot cle `chainsoc_test`.

### Etape 5 -- Envoi automatique vers Vault

Le script `send_to_vault.sh` fait tout en une seule execution : il recupere les logs, verifie si un log suspect est present, genere une alerte si necessaire, et envoie le log vers la machine Vault.

![Script send_to_vault.sh avec la logique de detection](screenshots/watcher_alert_detection_script.png)

### Etape 6 -- Generation d'alertes

Quand le script detecte un log suspect, il affiche `"ALERT: Suspicious log detected!"` et enregistre le log dans `alerts.txt` sur le Watcher.

### Etape 7 -- Automatisation avec cron

Le script `send_to_vault.sh` a ete ajoute au planificateur cron pour s'executer chaque minute :

```bash
* * * * * /home/hajar/send_to_vault.sh
```

Chaque minute, le systeme execute automatiquement : recuperation des logs, detection, creation d'alertes, envoi et stockage.

![Configuration cron pour l'execution automatique](screenshots/cron_automation_setup.png)

---

## 4. Scripts crees

| Script              | Fonction                                                    |
|---------------------|-------------------------------------------------------------|
| `check_logs.sh`     | Recupere les logs recents depuis Target via SSH              |
| `send_to_vault.sh`  | Detecte les logs suspects, genere des alertes, envoie vers Vault |

---

## 5. Test du systeme

Pour prouver que l'automatisation fonctionne, nous avons realise le test suivant :

**1.** Generer un log suspect sur Target :

```bash
logger "chainsoc_test_FINAL"
```

**2.** Attendre une minute (le temps que le cron s'execute).

**3.** Verifier les alertes sur Watcher :

```bash
cat alerts.txt
```

**4.** Verifier le stockage sur Vault :

```bash
cat vault_logs.txt
```

Le log suspect a ete detecte, l'alerte a ete creee, et le log a ete stocke sur Vault automatiquement. Ce test prouve que le systeme fonctionne de bout en bout sans intervention manuelle.

![Resultat du test : alertes detectees et logs stockes](screenshots/watcher_alert_detection_result.png)

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

Tout ce cycle s'execute automatiquement chaque minute.

---

## 7. Limites actuelles

Les composants suivants sont des implementations temporaires utilisees pour valider le fonctionnement du systeme :

| Composant         | Statut                               |
|-------------------|--------------------------------------|
| `alerts.txt`      | Stockage temporaire des alertes       |
| `vault_logs.txt`  | Stockage temporaire des logs sur Vault |

Ces fichiers texte ont permis de tester et prouver le bon fonctionnement de l'automatisation.

---

## 8. Next improvments

- **IPFS** -- Stockage decentralise des logs
- **Blockchain** -- Enregistrement des logs pour garantir leur integrite
- **Smart contracts** -- Verification automatique via des contrats intelligents
- **Dashboard** -- Interface de visualisation en temps reel

---

## Conclusion

ChainSOC demontre un prototype fonctionnel de SOC distribue. Le systeme collecte automatiquement les logs, detecte les activites suspectes, genere des alertes et stocke les donnees, le tout sans intervention manuelle. L'automatisation complete a ete validee par des tests concrets.
