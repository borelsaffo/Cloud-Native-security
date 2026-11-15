# Préparation à l'entretien --- Expert Sécurité OpenShift et conteneurs

## Objectif de la mission

La mission couvre la sécurisation de bout en bout d'une plateforme
cloud-native : architecture OpenShift, contrôle des accès, sécurité des
conteneurs, chaîne DevSecOps, segmentation réseau, sécurité au runtime,
maintien en condition de sécurité, audits et conformité.

Le profil attendu n'est donc pas uniquement celui d'un ingénieur
DevSecOps. Il doit aussi être capable de jouer le rôle de référent
sécurité, de conseiller les équipes techniques, de suivre les écarts et
de contribuer à la gestion des risques.

## Correspondance entre les exigences et les expériences à valoriser

  -----------------------------------------------------------------------
  Exigence                            Expériences et compétences à mettre
                                      en avant
  ----------------------------------- -----------------------------------
  OpenShift / Kubernetes              Kubernetes, EKS, environnements
                                      cloud-native et apprentissage des
                                      spécificités OpenShift

  RBAC / moindre privilège            RBAC Kubernetes, IRSA, Keycloak

  SCC / sécurité des workloads        Security Context, Kyverno,
                                      NetworkPolicies

  Hardening des clusters              Sécurisation EKS, Terraform,
                                      contrôles Kubernetes

  Gestion des secrets                 Vault, External Secrets

  Gestion des certificats             Venafi PKI, cert-manager, mTLS

  Sécurité CI/CD                      GitLab CI, Jenkins, SonarQube,
                                      Checkmarx

  Analyse des images                  Trivy et scans de vulnérabilités

  Admission Controllers               Kyverno et politiques d'admission

  Segmentation réseau                 NetworkPolicies, Istio, service
                                      mesh

  Sécurité runtime                    Wiz, CloudGuard et supervision

  Gestion des vulnérabilités          Trivy, Wiz, Checkov

  Audit et conformité                 Expérience d'audit IT tiers et
                                      contrôles de sécurité
  -----------------------------------------------------------------------

**Points forts à valoriser :** Kubernetes, DevSecOps, gestion des
secrets, segmentation réseau, politiques de sécurité, automatisation et
sécurité des communications.

**Points à approfondir :** spécificités OpenShift (SCC, architecture du
Control Plane, IPI/UPI), benchmark CIS OpenShift, Compliance Operator et
sécurité runtime avec Red Hat Advanced Cluster Security.

------------------------------------------------------------------------

# Domaine 1 --- Sécurisation des clusters OpenShift et du Control Plane

![Architecture générale d'un cluster OpenShift : Control Plane, etcd,
API Server et nœuds
workers](https://miro.medium.com/1%2ABvV6KtQ0yOW5GWMfwjBIrw.png)

*Illustration : architecture simplifiée d'un cluster OpenShift /
Kubernetes. Source : Medium, « A simplified overview of OCP ».*

## 1.1. Objectif

Sécuriser le cluster depuis son architecture jusqu'aux workloads, en
protégeant le plan de contrôle, les nœuds, les identités, les secrets et
les communications.

## 1.2. Composants à connaître

-   **API Server :** point d'entrée de l'administration et des
    opérations Kubernetes. Il faut contrôler l'authentification,
    l'autorisation, les accès réseau et les journaux d'audit.
-   **etcd :** base de données distribuée qui stocke l'état du cluster.
    Son accès doit être strictement limité ; les communications et les
    sauvegardes doivent être protégées et les données sensibles
    chiffrées au repos.
-   **Control Plane :** ensemble des composants qui pilotent l'état du
    cluster, notamment l'API Server, le scheduler, les controllers et
    etcd.
-   **Worker nodes :** nœuds qui exécutent les workloads. Leur système,
    les services du runtime et les accès administratifs doivent être
    durcis.
-   **Operators :** mécanismes de gestion du cycle de vie des composants
    OpenShift. Leurs permissions, leurs sources et leurs mises à jour
    doivent être maîtrisées.

## 1.3. Mesures de sécurisation

1.  Définir une architecture de sécurité et des zones de confiance.
2.  Appliquer le principe du moindre privilège avec RBAC.
3.  Restreindre les permissions des workloads avec les SCC.
4.  Éviter les conteneurs privilégiés et limiter les capabilities Linux.
5.  Sécuriser l'API Server et etcd.
6.  Mettre en place le chiffrement des secrets au repos lorsque requis.
7.  Organiser la rotation des secrets et des certificats.
8.  Appliquer les NetworkPolicies.
9.  Centraliser les logs d'audit.
10. Contrôler la conformité avec le benchmark CIS adapté à la version
    d'OpenShift.

## 1.4. RBAC et SCC : deux mécanismes complémentaires

-   **RBAC (Role-Based Access Control)** contrôle les actions qu'un
    utilisateur ou un ServiceAccount peut réaliser sur les ressources
    Kubernetes : lire, créer, modifier ou supprimer des objets.
-   **SCC (Security Context Constraints)** est un mécanisme propre à
    OpenShift qui contrôle les conditions de sécurité dans lesquelles un
    pod peut s'exécuter.

Les SCC peuvent notamment encadrer :

-   l'exécution en mode privilégié ;
-   l'utilisation du compte root et les plages d'UID ;
-   les Linux capabilities ;
-   les volumes autorisés ;
-   `hostNetwork`, `hostPID` et les autres accès à l'hôte ;
-   les contextes SELinux ;
-   les profils seccomp.

Exemples de commandes :

``` bash
oc get scc
oc describe scc restricted-v2
oc adm policy who-can use scc privileged
```

Pour attribuer une SCC à un ServiceAccount, vérifier d'abord la
politique de l'organisation et les droits nécessaires. Exemple de
syntaxe :

``` bash
oc adm policy add-scc-to-user <scc-name> -z <service-account> -n <namespace>
```

**Bonne pratique :** ne pas attribuer une SCC permissive pour contourner
rapidement un problème de déploiement. Identifier la permission
réellement nécessaire et appliquer le niveau de privilège minimal.

## 1.5. IPI et UPI

-   **IPI --- Installer-Provisioned Infrastructure :** l'installateur
    OpenShift provisionne l'infrastructure prise en charge par la
    plateforme.
-   **UPI --- User-Provisioned Infrastructure :** l'infrastructure est
    préparée et provisionnée par l'équipe avant l'installation
    d'OpenShift.

La différence a des conséquences sur les responsabilités d'exploitation,
l'automatisation, le durcissement et la gestion du cycle de vie de
l'infrastructure.

## 1.6. Exemple de réponse en entretien : « Comment sécurisez-vous un cluster OpenShift ? »

« Je commence par définir les zones de confiance et les responsabilités
d'administration. J'applique ensuite le moindre privilège avec RBAC et
je limite les capacités d'exécution des workloads avec les SCC. Je
sécurise l'API Server et etcd, je protège les secrets et les
certificats, puis je mets en place la segmentation réseau. Enfin, je
centralise les logs d'audit et je mesure régulièrement la conformité du
cluster par rapport au benchmark CIS applicable. Les écarts font l'objet
d'un plan de remédiation ou, si nécessaire, d'une dérogation
formellement validée. »

------------------------------------------------------------------------

# Domaine 2 --- Sécurité des conteneurs et chaîne DevSecOps

![Exemple de pipeline CI/CD avec analyse des vulnérabilités et signature
d'images](https://miro.medium.com/v2/resize%3Afit%3A1160/1%2A95wven9AHuJIWnC_FxxKPQ.png)

*Illustration : exemple de pipeline utilisant des contrôles de code,
Trivy et Cosign. Source : Medium, « Securing a GitHub Actions Pipeline
with Trivy, Snyk, and Cosign ».*

## 2.1. Objectif

Sécuriser l'ensemble du cycle de vie d'une image conteneur, depuis le
code source jusqu'à son exécution dans le cluster.

## 2.2. Pipeline de sécurité recommandé

  -----------------------------------------------------------------------
  Étape                               Contrôle ou outil possible
  ----------------------------------- -----------------------------------
  Développement                       SAST avec SonarQube ou Checkmarx

  Dépendances                         SCA et détection des vulnérabilités

  Infrastructure as Code              Checkov

  Construction de l'image             Dockerfile sécurisé, image de base
                                      maîtrisée

  Analyse de l'image                  Trivy

  Signature                           Cosign / Sigstore

  Admission dans le cluster           Kyverno ou Gatekeeper

  Surveillance à l'exécution          Red Hat Advanced Cluster Security
                                      (ACS), Falco ou solution
                                      équivalente
  -----------------------------------------------------------------------

## 2.3. Mesures à mettre en place

-   Utiliser des images de base maintenues, minimales et provenant de
    registres de confiance.
-   Éviter les secrets dans les Dockerfiles, les dépôts Git et les
    couches d'images.
-   Scanner le code, les dépendances, l'IaC et les images.
-   Définir des seuils de blocage selon la sévérité des vulnérabilités
    et le contexte d'exposition.
-   Produire des images identifiables et traçables ; éviter de dépendre
    uniquement de tags mutables.
-   Signer les images et vérifier leur signature avant admission.
-   Conserver la provenance des artefacts et, si possible, les éléments
    attestant leur construction.
-   Définir des règles d'admission pour empêcher les déploiements non
    conformes.
-   Surveiller les vulnérabilités découvertes après le déploiement.

## 2.4. Kyverno et OPA Gatekeeper

Les deux solutions permettent d'appliquer des politiques Kubernetes,
mais leur approche diffère :

-   **Kyverno :** politiques déclaratives généralement écrites en YAML ;
    permet notamment de valider, muter et générer des ressources.
-   **OPA Gatekeeper :** s'appuie sur Open Policy Agent et le langage
    Rego pour exprimer les règles de politique.

Les politiques peuvent par exemple imposer :

-   l'utilisation d'images provenant de registres autorisés ;
-   l'interdiction des conteneurs privilégiés ;
-   l'absence de `latest` comme tag d'image ;
-   la présence de limites CPU et mémoire ;
-   l'exécution avec un utilisateur non privilégié ;
-   la vérification de signatures d'images, selon les capacités et
    l'intégration retenues.

## 2.5. Exemple de réponse en entretien : « Comment empêcher le déploiement d'une image vulnérable ? »

« Je mets en place une défense en profondeur. La pipeline CI/CD analyse
d'abord le code, les dépendances et l'image avec les outils adaptés, par
exemple Trivy. Je définis des seuils de vulnérabilité bloquants en
tenant compte de la criticité et de l'exposition. Les images validées
sont publiées dans un registre privé et signées avec Cosign. Enfin, une
politique d'admission Kyverno ou Gatekeeper vérifie que les images
respectent les règles définies avant leur déploiement. La surveillance
continue permet ensuite de détecter les nouvelles vulnérabilités
affectant les images déjà en production. »

------------------------------------------------------------------------

# Domaine 3 --- Architecture et segmentation réseau

![Schéma de segmentation réseau Kubernetes entre namespaces et règles
ingress/egress](https://earthly.dev/.netlify/images?url=%2Fblog%2Fassets%2Fimages%2Fkubernetes-network-policies%2FT9DWs2n.png)

*Illustration : principe de contrôle des flux ingress et egress entre
namespaces. Source : Earthly, « Securing Kubernetes With Network
Policies ».*

## 3.1. Objectif

Réduire les communications latérales et limiter les possibilités de
déplacement d'un attaquant à l'intérieur d'un cluster multi-tenant.

## 3.2. Mécanismes à distinguer

-   **Isolation des namespaces :** séparation logique des ressources et
    des périmètres d'administration.
-   **RBAC :** contrôle des actions autorisées sur les ressources.
-   **NetworkPolicies :** contrôle des flux réseau au niveau IP/port,
    selon les sélecteurs de pods et de namespaces.
-   **Service Mesh :** gestion des communications interservices, avec
    des fonctions de sécurité, d'observabilité et de gestion du trafic.
-   **mTLS :** authentification mutuelle des services et chiffrement des
    communications.
-   **AuthorizationPolicies Istio :** contrôle des autorisations
    applicatives entre services dans le périmètre du service mesh.

## 3.3. Stratégie de segmentation

Pour isoler deux applications hébergées dans un même cluster :

1.  Créer des namespaces distincts selon les applications, les équipes
    ou les environnements.
2.  Définir des rôles RBAC adaptés à chaque périmètre.
3.  Appliquer une politique réseau restrictive, idéalement selon une
    approche Default Deny.
4.  Autoriser explicitement les flux nécessaires entre les composants.
5.  Contrôler les flux entrants et sortants (Ingress et Egress).
6.  Utiliser Istio et mTLS lorsque l'architecture le justifie pour
    sécuriser les communications interservices.
7.  Superviser les flux et réévaluer régulièrement les règles.

## 3.4. Exemple de NetworkPolicy

Cette politique autorise uniquement les pods identifiés comme `frontend`
à joindre le backend sur le port TCP 8080. Elle ne définit ici que le
contrôle Ingress du backend.

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

Pour une stratégie Default Deny complète, prévoir les politiques
d'isolation Ingress et Egress appropriées, puis ajouter les
autorisations nécessaires. Vérifier également que le plugin réseau
utilisé par le cluster prend en charge les NetworkPolicies.

## 3.5. NetworkPolicy et mTLS : quelle différence ?

-   **NetworkPolicy :** contrôle si un flux réseau est autorisé entre
    des workloads, généralement au niveau L3/L4.
-   **mTLS :** authentifie mutuellement les deux extrémités et chiffre
    la communication.
-   **Istio AuthorizationPolicy :** permet d'exprimer des règles
    d'autorisation plus fines, notamment en fonction de l'identité des
    services et du contexte de la requête.

Ces mécanismes sont complémentaires : autoriser un flux réseau ne
signifie pas que le service distant est authentifié au niveau
applicatif.

## 3.6. Exemple de réponse en entretien : « Comment isoler deux applications dans le même cluster ? »

« Je commence par séparer les applications dans des namespaces distincts
et par limiter les droits avec RBAC. J'applique ensuite des
NetworkPolicies en Default Deny, puis j'autorise uniquement les flux
nécessaires entre les composants. Si l'architecture utilise un service
mesh, je complète cette segmentation avec mTLS et des politiques
d'autorisation Istio. Enfin, je vérifie les flux réellement observés et
je maintiens les règles à jour. »

------------------------------------------------------------------------

# Domaine 4 --- Hardening et benchmark CIS

## 4.1. Objectif

Évaluer et améliorer le niveau de sécurité de la plateforme en
s'appuyant sur un référentiel de configuration reconnu, notamment le
benchmark CIS Red Hat OpenShift correspondant à la version déployée.

## 4.2. Principaux périmètres de contrôle

-   **Control Plane :** configuration et exposition de l'API Server,
    composants de contrôle et permissions.
-   **etcd :** accès, chiffrement, certificats et sauvegardes.
-   **Nœuds :** configuration système, accès administratifs, services et
    paramètres de sécurité.
-   **Authentification et autorisation :** configuration des identités,
    RBAC et comptes de service.
-   **Workloads :** privilèges, contextes de sécurité, images,
    ressources et politiques d'admission.
-   **Journalisation et audit :** collecte, conservation et analyse des
    événements.

## 4.3. Démarche d'audit CIS

1.  Identifier la version exacte d'OpenShift.
2.  Sélectionner le benchmark CIS correspondant à cette version et à
    l'environnement.
3.  Réaliser un état des lieux de la configuration.
4.  Exécuter les contrôles automatisés et compléter par des
    vérifications manuelles lorsque nécessaire.
5.  Identifier les écarts et les classer selon leur criticité et leur
    contexte.
6.  Évaluer les risques associés.
7.  Définir les plans de remédiation et les responsables.
8.  Formaliser les éventuelles dérogations.
9.  Réaliser un nouveau contrôle après correction.
10. Conserver les preuves et suivre les écarts dans le temps.

## 4.4. Outils à connaître

-   **Compliance Operator :** opérateur OpenShift permettant d'exécuter
    des contrôles de conformité à partir de profils adaptés.
-   **kube-bench :** outil de vérification de certains contrôles CIS
    Kubernetes ; sa pertinence et sa compatibilité doivent être
    vérifiées pour la version et la distribution utilisées.
-   **Red Hat Advanced Cluster Security :** plateforme de gestion des
    risques, des vulnérabilités et de la conformité
    Kubernetes/OpenShift.

## 4.5. Exemple de réponse en entretien : « Comment réalisez-vous un audit CIS sur OpenShift ? »

« Je commence par identifier la version de la plateforme et le benchmark
applicable. Je réalise ensuite les contrôles automatisés, notamment avec
le Compliance Operator lorsque le profil est adapté. Je consolide les
résultats, qualifie les écarts selon leur criticité et leur exposition,
puis je construis un plan de remédiation avec les équipes concernées.
Les exceptions sont documentées et validées selon le processus de
dérogation. Enfin, je relance les contrôles pour vérifier l'efficacité
des corrections et conserver les éléments de preuve. »

------------------------------------------------------------------------

# Domaine 5 --- Sécurité runtime et gestion des incidents

![Tableau de bord Red Hat Advanced Cluster Security pour
Kubernetes](https://www.redhat.com/rhdc/managed-files/ohc/q3_2021_operator_blog.png)

*Illustration : tableau de bord Red Hat Advanced Cluster Security.
Source : Red Hat.*

## 5.1. Objectif

Détecter les comportements anormaux ou malveillants pendant l'exécution
des conteneurs, y compris lorsque les images et les configurations ont
passé les contrôles initiaux.

## 5.2. Exemples de comportements suspects

-   Exécution inattendue d'un shell dans un conteneur.
-   Tentative d'élévation de privilèges.
-   Accès non autorisé au système de fichiers.
-   Connexion réseau inhabituelle.
-   Tentative de lecture de secrets.
-   Exécution d'un processus non prévu.
-   Accès inattendu aux ressources de l'hôte.

## 5.3. Outils et mécanismes

-   **Red Hat Advanced Cluster Security (ACS) :** gestion des
    vulnérabilités, des risques, de la conformité et des violations de
    politiques dans les environnements Kubernetes.
-   **Falco :** détection de comportements suspects à partir
    d'événements runtime.
-   **SIEM :** centralisation et corrélation des événements de sécurité,
    par exemple Splunk, QRadar ou Microsoft Sentinel.
-   **Wiz :** visibilité et analyse des risques dans les environnements
    cloud et cloud-native.

## 5.4. Processus de réponse à incident

1.  Détecter l'événement via les alertes runtime, les logs ou le SIEM.
2.  Qualifier l'alerte et déterminer les workloads, namespaces et actifs
    concernés.
3.  Évaluer la gravité, l'exposition et l'impact potentiel.
4.  Contenir l'incident : isoler le workload ou limiter les
    communications selon les procédures applicables.
5.  Préserver les éléments utiles à l'investigation.
6.  Identifier le vecteur d'attaque et les éventuelles compromissions
    associées.
7.  Corriger la cause : image, configuration, secret, permission ou
    vulnérabilité.
8.  Redéployer une version saine et vérifier le retour à un état
    maîtrisé.
9.  Documenter l'incident et mettre à jour les règles de détection.

## 5.5. Exemple de réponse en entretien : « Comment détecter une compromission d'un conteneur ? »

« Je combine la détection runtime, la journalisation et la corrélation
SIEM. Je définis des règles permettant de détecter les comportements
anormaux, comme l'exécution de processus inattendus, les élévations de
privilèges ou les connexions inhabituelles. Lorsqu'une alerte est
déclenchée, je qualifie l'événement, identifie le périmètre affecté,
applique les mesures de confinement, puis mène l'investigation et la
remédiation. Enfin, je documente l'incident et améliore les contrôles
pour éviter sa répétition. »

------------------------------------------------------------------------

# Domaine 6 --- Gouvernance, audit et conformité

## 6.1. Objectif

Faire appliquer la politique de sécurité, contrôler son respect,
instruire les dérogations et assurer le suivi des recommandations
d'audit. Ce volet demande une capacité à traduire les exigences de
sécurité en contrôles opérationnels et en plans d'action.

## 6.2. Contrôles périodiques de niveau 1

Un contrôle de niveau 1 est généralement un contrôle opérationnel
réalisé par les équipes responsables de l'activité ou du dispositif.

Exemples de contrôles :

-   Vérifier périodiquement les droits RBAC et les comptes de service.
-   Contrôler les workloads privilégiés et les SCC utilisées.
-   Vérifier l'application des NetworkPolicies.
-   Contrôler la rotation et la protection des secrets et certificats.
-   Vérifier le traitement des vulnérabilités critiques.
-   Contrôler la collecte des logs et le traitement des alertes.
-   Vérifier le suivi des dérogations et leur date d'expiration.

Pour chaque contrôle, définir une fréquence, un responsable, une
méthode, une preuve attendue, un résultat et un suivi des écarts.

## 6.3. Gestion des dérogations à la politique de sécurité

Une dérogation doit être encadrée et traçable. Elle ne doit pas devenir
une autorisation permanente implicite.

Démarche recommandée :

1.  Identifier l'exigence de sécurité non respectée.
2.  Comprendre le besoin métier ou technique à l'origine de la demande.
3.  Identifier les actifs, les menaces et les impacts potentiels.
4.  Évaluer le risque initial et le risque résiduel.
5.  Examiner les mesures compensatoires proposées.
6.  Définir un responsable, une durée de validité et une échéance.
7.  Faire valider la dérogation par l'autorité compétente.
8.  Suivre le plan de remédiation et réexaminer la dérogation avant son
    expiration.

**Exemple de réponse en entretien :**

« Je commence par identifier l'exigence de sécurité non respectée.
J'analyse le contexte, les actifs concernés, la menace et l'impact
potentiel. Je vérifie les mesures compensatoires proposées et j'évalue
le risque résiduel. Je formalise ensuite la dérogation avec un
responsable, une durée de validité et un plan de remédiation. La
validation revient à l'autorité compétente selon le processus de
gouvernance de l'entreprise. »

## 6.4. Participation aux audits

La contribution aux audits internes et externes comprend notamment :

-   la préparation du périmètre et des éléments de preuve ;
-   la présentation de l'architecture et des contrôles ;
-   la collecte des configurations, journaux et rapports ;
-   l'explication des écarts et des mesures compensatoires ;
-   la définition des actions correctives ;
-   le suivi des recommandations jusqu'à leur clôture.

## 6.5. Mise en conformité et amélioration continue

Pour chaque recommandation d'audit :

1.  Comprendre précisément le constat.
2.  Identifier la cause racine.
3.  Évaluer le risque associé.
4.  Définir une action corrective proportionnée.
5.  Affecter un responsable et une échéance.
6.  Tester et documenter la correction.
7.  Fournir une preuve de mise en conformité.
8.  Vérifier que l'écart ne réapparaît pas.

## 6.6. Guides, standards et conseil sécurité

Le référent sécurité doit également :

-   rédiger des guides de bonnes pratiques de développement sécurisé ;
-   produire des standards de configuration Kubernetes/OpenShift ;
-   conseiller les équipes sur les choix d'architecture ;
-   donner un avis sécurité lors de l'intégration de nouvelles solutions
    ;
-   proposer des améliorations de sécurité, de qualité, de performance
    et de coûts d'exploitation ;
-   assurer une veille technologique et réglementaire.

## 6.7. Lien avec la gestion des risques et ISO 27001

La démarche peut être rapprochée des principes d'ISO 27001 :

-   identifier les actifs et les exigences de sécurité ;
-   évaluer les risques ;
-   sélectionner et mettre en œuvre des mesures de traitement ;
-   conserver les preuves de fonctionnement des contrôles ;
-   suivre les non-conformités et les actions correctives ;
-   améliorer continuellement le système de management de la sécurité de
    l'information.

------------------------------------------------------------------------

# Synthèse --- Positionnement à défendre pendant l'entretien

Le positionnement le plus pertinent est celui d'un **référent sécurité
cloud-native / Kubernetes**, capable d'associer expertise technique,
automatisation et gouvernance.

  -----------------------------------------------------------------------
  Domaine                 Compétences à mettre en Préparation
                          avant                   
  ----------------------- ----------------------- -----------------------
  OpenShift et Control    Kubernetes, RBAC, SCC,  Approfondir les
  Plane                   secrets, certificats,   spécificités OpenShift
                          hardening               

  Conteneurs et DevSecOps GitLab CI, Jenkins,     Consolider la chaîne de
                          Trivy, Checkov,         confiance et
                          Kyverno, signature      l'admission
                          d'images                

  Segmentation réseau     NetworkPolicies, Istio, Être capable de
                          mTLS, isolation des     concevoir une politique
                          namespaces              Default Deny

  Hardening et CIS        Contrôles de            Approfondir CIS
                          configuration,          OpenShift et Compliance
                          conformité, remédiation Operator

  Runtime et incidents    Wiz, CloudGuard,        Approfondir ACS et
                          supervision, SIEM       Falco

  Gouvernance et          Audit IT, contrôles,    Préparer des exemples
  conformité              dérogations, risques,   concrets de contrôles
                          plans d'action          et d'audits
  -----------------------------------------------------------------------

## Point de vigilance sur l'expérience OpenShift

Si l'expérience opérationnelle est principalement basée sur
Kubernetes/EKS et que l'administration OpenShift a été pratiquée en
laboratoire ou en montée en compétence, il faut le présenter clairement.

Formulation possible :

« Mon expérience opérationnelle porte principalement sur les
environnements Kubernetes et cloud-native, notamment AWS EKS. J'ai
travaillé sur des problématiques directement transposables à OpenShift :
RBAC, NetworkPolicies, sécurité des workloads, gestion des secrets,
mTLS, politiques d'admission et DevSecOps. J'ai également approfondi les
spécificités OpenShift, notamment les SCC et l'administration de la
plateforme. Je maîtrise donc les fondamentaux de sécurité Kubernetes et
je suis capable de les décliner dans l'écosystème Red Hat OpenShift. »

## Priorités de révision

1.  SCC, RBAC et architecture du Control Plane OpenShift.
2.  IPI / UPI et hardening des nœuds.
3.  etcd, API Server, secrets et certificats.
4.  CIS OpenShift et Compliance Operator.
5.  Kyverno, Gatekeeper, Trivy et Cosign.
6.  NetworkPolicies, Default Deny, Istio et mTLS.
7.  Red Hat ACS, Falco, logs d'audit et SIEM.
8.  Contrôles périodiques, dérogations et suivi des recommandations
    d'audit.

------------------------------------------------------------------------

## Références visuelles utilisées

Les images intégrées dans ce document sont chargées depuis leurs sources
web d'origine. Une connexion Internet est donc nécessaire pour les
afficher dans certains lecteurs Markdown.

-   Architecture OpenShift : [Medium --- OCP
    architecture](https://medium.com/hybrid-cloud-engineering/transforming-websphere-to-open-liberty-on-openshift-an-operations-point-of-view-part-3-a-42b0129d12c4)
-   Pipeline DevSecOps : [Medium --- Trivy, Snyk and
    Cosign](https://medium.com/@aflousfatima/securing-a-github-actions-pipeline-with-trivy-snyk-and-cosign-ff8dc2710e52)
-   NetworkPolicies : [Earthly --- Kubernetes Network
    Policies](https://earthly.dev/blog/kubernetes-network-policies/)
-   Runtime security : [Red Hat --- Advanced Cluster
    Security](https://www.redhat.com/en/blog/the-advanced-cluster-security-operator-is-here.-what-you-need-to-know-and-how-to-get-started)
