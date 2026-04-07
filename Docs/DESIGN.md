# Projet PFE : Documentation du Design Fonctionnel

Ce document décrit le design du système pour chaque sprint, en détaillant les cas d'utilisation, les processus (séquences) et la structure des données (classes) sous forme de paragraphes descriptifs.

---

## Sprint 1 : Gestion des Utilisateurs

### 1.1 Diagramme des Cas d'Utilisation (Description)
Le système d'utilisateurs repose sur quatre acteurs principaux : l'Étudiant, l'Enseignant, le Chef de Département et l'Administrateur. L'Étudiant peut s'inscrire sur la plateforme, se connecter et consulter ses informations personnelles via son profil. L'Enseignant hérite des fonctionnalités de l'étudiant et possède en plus le droit de lister les étudiants pour ses besoins pédagogiques. Le Chef de Département, en tant que gestionnaire, a la responsabilité de gérer l'ensemble des utilisateurs de son département (ajout, modification, suppression). Enfin, l'Administrateur est le super-utilisateur ayant un accès global à la gestion de tous les comptes du système.

### 1.2 Processus d'Authentification (Séquence)
Lorsqu'un utilisateur tente de se connecter, il envoie ses identifiants (email et mot de passe) via l'interface. Le contrôleur d'authentification reçoit cette demande et interroge la base de données pour trouver l'utilisateur correspondant à l'email fourni. Si l'utilisateur est trouvé, le système compare le mot de passe saisi avec la version hachée stockée en base. Si la vérification réussit, un jeton de sécurité (JWT) est généré et renvoyé à l'utilisateur avec ses informations de profil. En cas d'échec (email inexistant ou mot de passe incorrect), le système renvoie une erreur d'authentification sécurisée.

### 1.3 Structure des Données Utilisateurs (Classe)
L'entité "Utilisateur" est le pilier du système. Elle stocke les informations d'identité (nom, prénom, email, mot de passe haché) et définit le rôle de l'individu (ADMIN, STUDENT, TEACHER, etc.). Pour les étudiants, des champs spécifiques comme le numéro d'inscription, l'identifiant étudiant et le lien vers une classe (classId) sont utilisés. Le modèle gère également l'état du compte (actif ou non) et enregistre automatiquement les dates de création et de mise à jour de chaque profil.

---

## Sprint 2 : Gestion des Emplois du Temps

### 2.1 Diagramme des Cas d'Utilisation (Description)
La gestion des emplois du temps implique l'Étudiant, l'Enseignant et le Chef de Département. L'Étudiant et l'Enseignant peuvent consulter leur planning hebdomadaire respectif. L'Enseignant a également accès à la liste des cours qu'il dispense. Le Chef de Département possède les droits d'administration académique : il peut créer et modifier les modules de cours, gérer les salles de classe disponibles et organiser les séances en affectant un enseignant, une classe et une salle à un créneau horaire précis.

### 2.2 Consultation de l'Emploi du Temps (Séquence)
Lorsqu'un étudiant souhaite voir son planning, le système identifie d'abord la classe à laquelle il appartient via son profil. Il recherche ensuite dans la base de données du département la correspondance entre cette classe et son identifiant externe (issu du système de planification). Une fois la classe localisée, le contrôleur récupère toutes les séances programmées pour cette classe, les trie par jour de la semaine et par créneau horaire (slot), puis les transmet à l'interface de l'étudiant pour affichage.

### 2.3 Structure de la Gestion Académique (Classe)
Le design repose sur quatre entités interconnectées. Le "Département" regroupe les enseignants et les classes. Le "Cours" (ou Module) définit le contenu pédagogique, le volume horaire et le niveau. La "Salle" caractérise les lieux de cours par leur bâtiment et leur capacité. Enfin, la "Séance" (Session) fait le lien entre ces éléments : elle associe un cours, un enseignant, une salle et une classe à un moment précis (jour et créneau), tout en précisant le type de session (Cours, TD ou TP).

---

## Sprint 3 : Gestion des Absences

### 3.1 Diagramme des Cas d'Utilisation (Description)
Ce module permet le suivi de l'assiduité des étudiants. L'Étudiant peut consulter son historique d'absences pour vérifier son statut et éventuellement fournir des justifications. L'Enseignant est l'acteur principal de la saisie : il peut marquer la présence ou l'absence d'un étudiant individuellement ou effectuer une saisie en masse pour toute une classe lors d'une séance. Il a aussi le droit de modifier ou supprimer un enregistrement en cas d'erreur. Le Chef de Département supervise l'ensemble des données de présence de son département.

### 3.2 Marquage de Présence en Masse (Séquence)
Pour gagner du temps, l'enseignant utilise la fonctionnalité de saisie groupée. Il sélectionne le cours, la date et la séance concernée. Le système lui présente la liste des étudiants inscrits dans cette classe. L'enseignant coche le statut de chaque étudiant (Présent par défaut, Absent, en Retard). Une fois validée, la liste est envoyée au serveur qui transforme chaque ligne en un document de présence individuel. Ces documents sont insérés simultanément dans la base de données, et un message de confirmation indiquant le nombre d'enregistrements créés est renvoyé à l'enseignant.

### 3.3 Structure des Données de Présence (Classe)
L'entité "Présence" enregistre chaque événement d'assiduité. Elle contient les références vers l'étudiant concerné et l'enseignant qui a effectué l'appel. Elle précise le nom du cours, la date, la durée de la séance et le type (Cours, examen, etc.). Le statut (PRÉSENT, ABSENT, EN RETARD, EXCUSÉ) est l'information centrale. Le modèle prévoit également des champs pour la justification (texte) et un indicateur booléen précisant si l'absence a été validée comme justifiée par l'administration.

---

## Sprint 4 : Gestion des Notes

### 4.1 Diagramme des Cas d'Utilisation (Description)
La gestion des notes ferme le cycle pédagogique. L'Étudiant consulte ses résultats par semestre et peut déposer une réclamation s'il constate une anomalie. L'Enseignant saisit les notes, soit une par une, soit en important un fichier Excel contenant les résultats de toute une classe. Il peut aussi supprimer une note erronée qu'il a précédemment attribuée. Le Chef de Département, en plus de consulter les notes, est responsable du traitement des réclamations déposées par les étudiants (acceptation ou rejet avec commentaire).

### 4.2 Importation de Notes via Excel (Séquence)
Le processus d'importation commence par le téléchargement d'un fichier Excel par l'enseignant, accompagné des métadonnées (nom du cours, semestre, type d'évaluation). Le serveur lit le fichier et extrait les données ligne par ligne. Pour chaque ligne, il vérifie si l'identifiant étudiant existe en base de données et si la note est valide (entre 0 et 20). Une fois toutes les lignes validées, le système crée massivement les entités "Note" correspondantes. Le résultat de l'opération, incluant le nombre de notes créées et les éventuelles erreurs rencontrées (étudiant inconnu, format incorrect), est renvoyé à l'enseignant.

### 4.3 Structure des Notes et Réclamations (Classe)
Deux entités gèrent ce module. La "Note" (Grade) stocke le résultat chiffré, le coefficient, le type d'examen (DS, Exam, TP) et les références de l'étudiant, de l'enseignant et du module. La "Réclamation" (Complaint) est liée à une note spécifique. Elle contient le motif rédigé par l'étudiant, le statut de la demande (En attente, Acceptée, Rejetée) et la réponse finale apportée par le Chef de Département, ainsi que la date et l'auteur de la résolution.
