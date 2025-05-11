
# 🔐 Audit de sécurité – Vulnerable Light App

## 📌 Objectifs

- Identifier les vulnérabilités de sécurité dans le code source de l’application.
- Evaluer l’impact et la gravité de chaque vulnérabilité.
- Valider l’exploitabilité via des tests d’intrusion.
- Proposer des recommandations correctives.

---

## 🧪 Méthodologie

- **Analyse statique** : lecture manuelle du code source (C#, .NET).
- **Analyse dynamique** : tests d’exploitation sur une instance locale de l’application.
- **Référentiel** : classification selon les références MITRE CWE.

---

## 📋 Bilan des vulnérabilités

### 🎯 Vulnérabilités exploitées avec succès

- OS Command Injection (CWE-78)
- Désérialisation non sécurisée (CWE-502)
- JWT forgé avec alg=none + secret exposé (CWE-1270 / CWE-798)
- Upload de fichier malveillant (CWE-434)
- Path Traversal & LFI (CWE-22 / CWE-829)
- XML External Entity (CWE-611)
- SSRF (CWE-918)
- Injection XSS stockée dans logs HTML (CWE-79)
- Injection SQL (CWE-89)
- Injection XML (CWE-91)
- Remote File Inclusion (CWE-98)
- Business Logic Error (CWE-840)
- IDOR (CWE-639)
- Buffer Overflow (CWE-787)
- Journalisation de credentials (CWE-532)
- Mauvaises politiques JWT (CWE-213)
- Contrôle d'accès incorrect (CWE-284)

---

## 🧪 Tests d'intrusion

### A. Authentification avec JWT

Nous avons utilisé un token JWT valide pour s’authentifier.

**SCREENSHOT ICI**

---

### B. Exploitation de l’IDOR (CWE-639)

Modification de l’ID dans l’URL : accès aux données d'autres employés.

**SCREENSHOT ICI**

---

### C. Path Traversal (CWE-22)

Accès au fichier source `Controller.cs` via contournement des protections.

**SCREENSHOT ICI**

---

### D. Command Injection (CWE-78)

Injection via `; whoami` dans un champ censé être un nom de domaine.

**SCREENSHOT ICI**

---

### E. Désérialisation (CWE-502)

Payload JSON contenant un objet `System.Diagnostics.Process`.

**SCREENSHOT ICI**

---

### F. Exécution de code C# (CWE-94)

Tentative d’évaluation dynamique. Résultat : erreur de compilation confirmant la vulnérabilité.

**SCREENSHOT ICI**

---

### G. Upload de fichier malveillant (CWE-434)

Fichier `shell.php.svg` uploadé et accessible.

**SCREENSHOT ICI**

---

### H. JWT forgé (CWE-1270 / CWE-798)

Token avec alg `none` accepté ou signé avec secret trouvé.

**SCREENSHOT ICI**

---

### I. XXE (CWE-611)

Fichier XML avec entité externe lisant `appsettings.json`.

**SCREENSHOT ICI**

---

### J. SSRF (CWE-918)

Tentative de requête vers `localhost:8080`. Erreur confirmant l'accès côté serveur.

**SCREENSHOT ICI**

---

### K. Buffer Overflow (CWE-787)

Payload long provoquant une coupure de la connexion TLS.

**SCREENSHOT ICI**

---

### L. Business Logic Error (CWE-840)

Dépassement d’entier entraînant une valeur négative inattendue.

**SCREENSHOT ICI**

---

### M. Logs avec credentials en clair (CWE-532)

Injection de commande pour lire les logs contenant des mots de passe.

**SCREENSHOT ICI**

---

### N. XSS dans les logs (CWE-79)

Payload `<img onerror=alert(document.cookie)>` exécuté côté admin.

**SCREENSHOT ICI**

---

### O. Injection SQL (CWE-89)

Tentative de login avec `' OR 1=1 --` provoquant une erreur SQL révélatrice.

**SCREENSHOT ICI**

---

### P. XML Injection (CWE-91)

Payload XML contenant une structure modifiée acceptée et interprétée.

**SCREENSHOT ICI**

---

### Q. Remote File Inclusion (CWE-98)

Fichier SVG inclus dans une réponse côté serveur.

**SCREENSHOT ICI**

---

## 📊 Impact et criticité des vulnérabilités

| Vulnérabilité                        | CWE             | Gravité   | Exploitabilité | Impact potentiel                                        | Priorité    |
|-------------------------------------|-----------------|-----------|----------------|----------------------------------------------------------|-------------|
| Injection de commandes OS           | CWE-78          | Critique  | Facile         | Exécution root, compromission complète du serveur       | 🔴 Haute    |
| JWT vulnérable + secret exposé      | CWE-1270 / 798  | Critique  | Facile         | Auth bypass, élévation de privilège                     | 🔴 Haute    |
| Désérialisation non sécurisée       | CWE-502         | Critique  | Moyenne        | Exécution de code arbitraire                            | 🔴 Haute    |
| Upload de fichiers dangereux        | CWE-434         | Critique  | Moyenne        | Inclusion de fichiers malveillants                      | 🔴 Haute    |
| Remote File Inclusion               | CWE-98          | Haute     | Moyenne        | Inclusion de contenu arbitraire                         | 🟠 Haute    |
| Path Traversal / LFI                | CWE-22 / 829    | Haute     | Facile         | Lecture de fichiers sensibles, code source              | 🟠 Haute    |
| XXE / SSRF                          | CWE-611 / 918   | Haute     | Moyenne        | Accès fichiers locaux ou requêtes internes              | 🟠 Haute    |
| Code C# dynamique (eval)            | CWE-94          | Haute     | Moyenne        | Injection de code dans l'exécution                      | 🟠 Haute    |
| Buffer Overflow                     | CWE-787         | Moyenne   | Moyenne        | Crash application, instabilité                          | 🟡 Moyenne  |
| Erreur de logique métier            | CWE-840         | Moyenne   | Moyenne        | Bypass des calculs, facturation corrompue               | 🟡 Moyenne  |
| IDOR / Accès non autorisé           | CWE-639 / 284   | Moyenne   | Facile         | Accès à des données d'autres utilisateurs               | 🟡 Moyenne  |
| Injection SQL (DataSet)             | CWE-89          | Moyenne   | Faible         | Contournement logique partiel                           | 🟡 Moyenne  |
| Injection XML                       | CWE-91          | Moyenne   | Moyenne        | Perturbation des structures XML                         | 🟡 Moyenne  |
| XSS dans les logs HTML              | CWE-79          | Moyenne   | Facile         | Exécution JS côté admin (session hijack possible)       | 🟡 Moyenne  |
| Infos sensibles dans les logs       | CWE-532 / 200   | Moyenne   | Facile         | Récupération des credentials                            | 🟢 Moyenne  |
| Mauvaises politiques d’auth JWT     | CWE-213         | Moyenne   | Facile         | Acceptation de tokens falsifiés                         | 🟢 Moyenne  |

---

## ✅ Recommandations générales

- Appliquer des **validations strictes** sur toutes les entrées utilisateur.
- Mettre en place une **authentification robuste** (limite de tentatives, mot de passe fort).
- Supprimer les **secrets en clair** du code source.
- Restreindre les droits d’exécution des fichiers uploadés.
- Implémenter une **politique de journalisation sécurisée**.
- Éviter les fonctions d’exécution dynamique (`eval`, `stackalloc`, etc.).
- Adopter une approche **DevSecOps** avec des outils d’analyse continue.

---

## 🧾 Conclusion

Cet audit a révélé plusieurs vulnérabilités critiques qui, si elles étaient exploitées dans un environnement réel, pourraient compromettre l’intégrité, la confidentialité et la disponibilité de l’application. La majorité des failles ont pu être exploitées avec succès dans un environnement de test contrôlé, ce qui confirme leur réalisme.

L'application nécessite une refonte ou un renforcement significatif de ses mécanismes de sécurité. Les vulnérabilités critiques doivent être corrigées en priorité. Cet audit démontre l’importance de la sécurité dès la conception, et l’utilité d’un processus d’audit continu dans le cycle de vie logiciel.

---

🛡️ *Audit réalisé par Bryan MARLOT – ESGI 24-25 – 2025*
