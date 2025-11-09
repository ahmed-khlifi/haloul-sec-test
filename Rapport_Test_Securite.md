# Rapport de Tests de Sécurité
**Généré le :** 9 novembre 2025

---

## Captures d'écran AutoScan-Front

### 1. En-tête Anti-clickjacking Manquant
**Catégorie :** Protection Clickjacking - En-tête de Sécurité Manquant

![En-tête Anti-clickjacking Manquant](autoScan-front/frontend-01.png)

**Problème :**
L'application ne possède pas l'en-tête X-Frame-Options ou Content-Security-Policy avec la directive frame-ancestors. Cela permet d'intégrer la page dans des iframes, rendant possible les attaques de clickjacking.

**Raison :**
Sans en-têtes anti-clickjacking, les attaquants peuvent charger l'application dans une iframe invisible et inciter les utilisateurs à cliquer sur des éléments cachés, conduisant potentiellement à des actions non autorisées.

**Solution :**
Ajouter l'en-tête `X-Frame-Options: DENY` ou `X-Frame-Options: SAMEORIGIN`. Alternativement, utiliser Content-Security-Policy avec la directive `frame-ancestors 'none'` ou `frame-ancestors 'self'` pour prévenir les attaques par framing.

---

### 2. CSP : Échec de Définir une Directive Sans Fallback
**Catégorie :** Content Security Policy - Configuration de Directive Manquante

![Problème Configuration CSP](autoScan-front/frontend-02.png)

**Problème :**
La Content Security Policy (CSP) manque de directives critiques sans configurations de secours appropriées. Cela affaiblit la posture de sécurité et peut permettre à certains types d'attaques de réussir.

**Raison :**
Lorsque les directives CSP ne sont pas correctement définies, le navigateur ne peut pas appliquer de restrictions de sécurité sur les sources de contenu, laissant l'application vulnérable aux attaques XSS et d'injection de données.

**Solution :**
Définir explicitement toutes les directives CSP nécessaires (script-src, style-src, img-src, etc.). Inclure une directive default-src comme fallback. Utiliser des politiques strictes comme 'self' et éviter 'unsafe-inline' et 'unsafe-eval' autant que possible.

---


### 3. Mauvaise Configuration Inter-domaines (CORS)
**Catégorie :** Mauvaise Configuration Cross-Origin Resource Sharing

![Mauvaise Configuration CORS](autoScan-front/frontend-04.png)

**Problème :**
L'application a une configuration Cross-Origin Resource Sharing (CORS) inappropriée, permettant potentiellement à des domaines non autorisés d'accéder aux ressources ou utilisant des paramètres trop permissifs comme `Access-Control-Allow-Origin: *`.

**Raison :**
Une mauvaise configuration CORS peut exposer des données sensibles à des sites web malveillants, permettre des appels API non autorisés depuis des origines non fiables et contourner les protections de la Same-Origin Policy.

**Solution :**
Configurer CORS correctement en spécifiant les origines autorisées exactes au lieu de wildcards. Utiliser `Access-Control-Allow-Credentials: true` uniquement avec des origines spécifiques, jamais avec `*`. Valider et whitelister explicitement les domaines de confiance.

---

### 4. Fuite d'Information via l'En-tête "X-Powered-By"
**Catégorie :** Divulgation d'Information - Exposition de la Technologie Serveur

![Fuite En-tête X-Powered-By](autoScan-front/frontend-05.png)

**Problème :**
Le serveur révèle des informations sur la pile technologique via l'en-tête X-Powered-By (par ex. "X-Powered-By: Express", "PHP/7.4", etc.), exposant les détails du framework et de la version aux attaquants potentiels.

**Raison :**
La divulgation de la technologie et des versions du serveur aide les attaquants à identifier les vulnérabilités connues spécifiques à ces frameworks et à planifier des attaques ciblées contre des composants obsolètes ou vulnérables.

**Solution :**
Supprimer ou masquer l'en-tête X-Powered-By. Dans Express.js utiliser `app.disable('x-powered-by')`. Dans PHP définir `expose_php = Off` dans php.ini. Configurer les serveurs web (Nginx/Apache) pour masquer les informations de version.

---

### 5. En-tête X-Content-Type-Options Manquant
**Catégorie :** Sécurité MIME-Type - Protection d'En-tête Manquante

![X-Content-Type-Options Manquant](autoScan-front/frontend-06.png)

**Problème :**
L'application ne définit pas l'en-tête X-Content-Type-Options. Cela permet aux navigateurs d'effectuer du MIME-type sniffing, interprétant potentiellement les fichiers différemment du type déclaré.

**Raison :**
Sans cet en-tête, les navigateurs peuvent exécuter le contenu différemment de ce qui était prévu. Par exemple, un fichier texte pourrait être interprété comme JavaScript, ou une image comme HTML, conduisant à des vulnérabilités XSS par confusion de type de contenu.

**Solution :**
Ajouter l'en-tête `X-Content-Type-Options: nosniff` à toutes les réponses HTTP. Cela force les navigateurs à suivre strictement le Content-Type déclaré et prévient les attaques par MIME-sniffing.

---

## Captures d'écran ScanBackend

### 7. actuatorTest.png
**Catégorie :** Divulgation d'Information - Exposition de Spring Boot Actuator

<img src="scanBackend/actuatorTest.png" width="600" alt="Exposition Actuator">

**Problème :**
La capture d'écran montre des endpoints Spring Boot Actuator exposés qui révèlent des informations sensibles sur l'application et des capacités de gestion.

**Déscription :**
Api actuator securisé pour cacher les configuration et les variable de connexion de la base de donnée , ainsi que l admin de l applicagion necessite une authentification pour accede a ces end points
---

### 8. backend-graph-end points.png
**Catégorie :** Énumération d'API - Découverte d'Endpoints GraphQL

<img src="scanBackend/backend-graph-end points.png" width="600" alt="Endpoints GraphQL">

**Problème :**
Cette capture d'écran affiche la structure des endpoints de l'API GraphQL, montrant potentiellement le schéma API complet et les opérations disponibles.

**Raison :**
L'introspection GraphQL est activée, permettant aux attaquants de cartographier toute la surface de l'API et d'identifier des cibles d'attaque potentielles.

**Solution :**
Désactiver l'introspection GraphQL dans les environnements de production. Implémenter la limitation de profondeur de requête et l'analyse de complexité pour prévenir les attaques d'épuisement de ressources.

---

### 9. backend-schema-shown.png
**Catégorie :** Divulgation d'Information - Exposition du Schéma de Base de Données

<img src="scanBackend/backend-schema-shown.png" width="600" alt="Exposition du Schéma">

**Problème :**
La capture d'écran révèle la structure du schéma de la base de données backend, exposant les noms de tables, les relations et l'organisation des données.

**Raison :**
La divulgation d'informations sur le schéma aide les attaquants à créer des attaques SQL injection ciblées et à comprendre les relations de données pour l'escalade de privilèges.

**Solution :**
Implémenter une gestion appropriée des erreurs pour prévenir les fuites de schéma. Utiliser des frameworks ORM avec des requêtes paramétrées et désactiver les messages d'erreur détaillés en production.

---

### 10. backend-test1.png
**Catégorie :** Tests de Sécurité Backend - Évaluation Initiale des Vulnérabilités

<img src="scanBackend/backend-test1.png" width="600" alt="Test Initial Backend">

**Problème :**
Cette capture d'écran montre les tests backend initiaux, identifiant probablement des faiblesses d'authentification, d'autorisation ou de validation des entrées.

**Raison :**
Les systèmes backend contiennent souvent des failles de logique métier, des contrôles d'accès défaillants ou une validation des entrées insuffisante qui peuvent être exploités.

**Solution :**
Implémenter des vérifications robustes d'authentification et d'autorisation. Utiliser des frameworks comme Spring Security avec contrôle d'accès basé sur les rôles (RBAC) et valider toutes les entrées côté serveur.

---

### 11. deepQueryBackend.png
**Catégorie :** Déni de Service - Attaque par Requête Profonde

<img src="scanBackend/deepQueryBackend.png" width="600" alt="Attaque Requête Profonde">

**Problème :**
Cette capture d'écran démontre une attaque par requête profonde contre le backend, causant potentiellement une dégradation des performances ou une interruption de service.

**Raison :**
GraphQL permet des requêtes imbriquées qui peuvent augmenter exponentiellement le temps de traitement et la consommation de mémoire, conduisant à des conditions de DoS.

**Solution :**
Implémenter la limitation de profondeur de requête (profondeur max : 3-5 niveaux). Ajouter l'analyse de complexité de requête et la limitation de taux pour prévenir les attaques d'épuisement de ressources.

---

### 12. deepQueryExtreme.png
**Catégorie :** Déni de Service - Exploitation Extrême par Requête Profonde

<img src="scanBackend/deepQueryExtreme.png" width="600" alt="Requête Profonde Extrême">

**Problème :**
Cette capture d'écran montre une attaque par requête profonde extrême, démontrant l'exploitation maximale des vulnérabilités de requêtes imbriquées.

**Raison :**
Sans contrôles appropriés de complexité de requête, les attaquants peuvent créer des requêtes avec une imbrication extrême qui submergent les ressources du serveur.

**Solution :**
Utiliser des bibliothèques comme `graphql-depth-limit` et `graphql-validation-complexity`. Définir une profondeur maximale de requête à 5 et implémenter des mécanismes de timeout pour les requêtes longues.

---

### 13. introcepction_query.png
**Catégorie :** Divulgation d'Information - Introspection GraphQL

<img src="scanBackend/introcepction_query.png" width="600" alt="Introspection GraphQL">

**Problème :**
Cette capture d'écran montre des requêtes d'introspection GraphQL révélant le schéma API complet, les types et les mutations disponibles.

**Raison :**
L'introspection est une fonctionnalité GraphQL qui permet à quiconque d'interroger et de découvrir toute la structure de l'API sans authentification.

**Solution :**
Désactiver l'introspection en production : `graphql.playground.enabled=false` et `graphql.introspection.enabled=false`. Utiliser le schema stitching uniquement pour les clients autorisés.

---

### 14. jwtTokenTest.png
**Catégorie :** Contournement d'Authentification - Vulnérabilité de Token JWT

<img src="scanBackend/jwtTokenTest.png" width="600" alt="Test Token JWT">

**Problème :**
Cette capture d'écran démontre une manipulation de token JWT ou un contournement de validation, permettant un accès non autorisé aux ressources protégées.

**Raison :**
Une implémentation JWT faible peut permettre la falsification de tokens, des attaques de confusion d'algorithme ou l'acceptation de tokens non signés.

**Solution :**
Utiliser des algorithmes de signature forts (RS256 au lieu de HS256). Valider rigoureusement la signature, l'expiration et les revendications du token. Ne jamais faire confiance aux en-têtes d'algorithme fournis par le client.

---

### 15. sqlinjectionRowAdd.png
**Catégorie :** Injection SQL - Manipulation de Données

<img src="scanBackend/sqlinjectionRowAdd.png" width="600" alt="Injection SQL Ajout Ligne">

**Problème :**
Cette capture d'écran montre une attaque par injection SQL réussie ajoutant des lignes non autorisées à la base de données.

**Raison :**
L'application concatène directement les entrées utilisateur dans les requêtes SQL sans sanitisation ou paramétrage approprié.

**Solution :**
Utiliser exclusivement des instructions préparées et des requêtes paramétrées. Implémenter des frameworks ORM comme JPA/Hibernate avec une configuration appropriée. Ne jamais concaténer les entrées utilisateur dans SQL.

---

### 16. sqlInjectionUnion.png
**Catégorie :** Injection SQL - Extraction de Données par UNION

<img src="scanBackend/sqlInjectionUnion.png" width="600" alt="Injection SQL Union">

**Problème :**
Cette capture d'écran démontre une injection SQL basée sur UNION extrayant des données sensibles des tables de base de données.

**Raison :**
Le manque de validation des entrées permet aux attaquants d'ajouter des instructions UNION SELECT pour extraire des données d'autres tables ou colonnes.

**Solution :**
Implémenter une validation stricte des entrées et une liste blanche de caractères autorisés. Utiliser des frameworks ORM avec des instructions préparées. Appliquer le principe du moindre privilège pour les comptes de base de données.

---

## Fichiers Supplémentaires

### 17. validator.png
**Catégorie :** Tests de Validation des Entrées

<img src="validator.png" width="600" alt="Test Validateur">

**Problème :**
Cette capture d'écran montre probablement les résultats des tests de validation, démontrant possiblement des contournements dans les mécanismes de validation des entrées.

**Raison :**
La validation côté client seule est insuffisante car elle peut être contournée. La validation côté serveur doit vérifier toutes les entrées.

**Solution :**
Implémenter une validation complète côté serveur pour toutes les entrées utilisateur. Utiliser des frameworks de validation comme Hibernate Validator avec des contraintes personnalisées pour les règles complexes.

---

## Résumé des Résultats

### Vulnérabilités Critiques (Action Immédiate Requise)
1. **Injection SQL** - Plusieurs vecteurs d'attaque démontrés
2. **Vulnérabilités Token JWT** - Contournement d'authentification possible
3. **Introspection GraphQL Activée** - Divulgation complète de l'API

### Problèmes de Haute Gravité
1. **Exposition Spring Boot Actuator** - Endpoints de gestion accessibles
2. **Attaques DoS par Requêtes Profondes** - Interruption de service possible
3. **Divulgation du Schéma de Base de Données** - Fuite d'informations

### Problèmes de Gravité Moyenne
1. **Faiblesses de Validation des Entrées** - Diverses techniques de contournement
2. **Lacunes de Sécurité Frontend** - Vulnérabilités côté client

### Actions Prioritaires Recommandées

1. **Immédiat (Dans les 24 heures) :**
   - Désactiver l'introspection GraphQL en production
   - Implémenter la prévention d'injection SQL avec des instructions préparées
   - Sécuriser la validation des tokens JWT

2. **Court terme (Dans 1 semaine) :**
   - Implémenter la limitation de profondeur et de complexité des requêtes
   - Sécuriser les endpoints Spring Boot Actuator
   - Ajouter une validation complète des entrées

3. **Moyen terme (Dans 1 mois) :**
   - Effectuer une revue de code de sécurité
   - Implémenter des tests de sécurité automatisés dans le CI/CD
   - Ajouter un pare-feu d'application web (WAF)

---

## Conclusion

Les tests de sécurité ont révélé plusieurs vulnérabilités critiques dans les composants frontend et backend. Les problèmes les plus graves concernent l'injection SQL, le contournement d'authentification et la divulgation d'informations via l'introspection GraphQL. Une remédiation immédiate est nécessaire pour prévenir les violations de données et les interruptions de service.

**Prochaines Étapes :**
- Prioriser les corrections en fonction des niveaux de gravité
- Implémenter les tests de sécurité dans le pipeline de développement
- Planifier des tests de pénétration de suivi après la remédiation
- Fournir une formation en sécurité à l'équipe de développement

---

*Ce rapport est basé sur l'analyse des captures d'écran et devrait être vérifié avec des outils de test de vulnérabilité réels et une revue de code manuelle.*
