# Projet PFE : Documentation du Design Global (Mermaid UML)

Ce document présente la vision globale du système à travers des diagrammes de cas d'utilisation, de séquences et de classes couvrant l'ensemble des modules.

---

## 1. Diagramme des Cas d'Utilisation Global (avec <<include>> et <<extend>>)

Ce diagramme illustre les interactions entre les acteurs et les fonctionnalités du système, en précisant les relations de dépendance.

```mermaid
graph TD
    %% Acteurs
    Student[Étudiant]
    Teacher[Enseignant]
    Chef[Chef de Département]
    Admin[Administrateur]

    %% Cas d'Utilisation - Auth & Profil
    UC_Login(Se connecter)
    UC_Register(S'inscrire)
    UC_Profile(Gérer son profil)

    %% Cas d'Utilisation - Académique
    UC_Schedule(Consulter l'emploi du temps)
    UC_ManageAcad(Gérer Séances & Cours)
    UC_Dept(Gérer Départements & Classes)

    %% Cas d'Utilisation - Évaluation
    UC_Attendance(Gérer la présence)
    UC_Justify(Justifier une absence)
    UC_Grades(Gérer les notes)
    UC_Excel(Importer via Excel)
    UC_Complaint(Déposer une réclamation)
    UC_Resolve(Traiter les réclamations)

    %% Relations d'Héritage d'Acteurs
    Teacher --> Student
    Chef --> Teacher
    Admin --> Chef

    %% Relations de Cas d'Utilisation (Include / Extend)
    UC_Profile -.->|"<<include>>"| UC_Login
    UC_Schedule -.->|"<<include>>"| UC_Login
    UC_Attendance -.->|"<<include>>"| UC_Login
    UC_Grades -.->|"<<include>>"| UC_Login

    UC_Justify -.->|"<<extend>>"| UC_Attendance
    UC_Excel -.->|"<<extend>>"| UC_Grades
    UC_Excel -.->|"<<extend>>"| UC_Attendance
    UC_Complaint -.->|"<<extend>>"| UC_Grades

    %% Affectations Acteurs -> UC
    Student --> UC_Register
    Student --> UC_Login
    Student --> UC_Profile
    Student --> UC_Schedule
    Student --> UC_Complaint

    Teacher --> UC_Attendance
    Teacher --> UC_Grades

    Chef --> UC_ManageAcad
    Chef --> UC_Dept
    Chef --> UC_Resolve

    %% Notes explicatives
    note_inc[<<include>> : Fonctionnalité obligatoire pour l'action]
    note_ext[<<extend>> : Optionnel ou conditionnel]

    style note_inc fill:#f9f,stroke:#333,stroke-dasharray: 5 5
    style note_ext fill:#bbf,stroke:#333,stroke-dasharray: 5 5
```

---

## 2. Diagramme de Séquence Global (Cycle Pédagogique)

Ce diagramme illustre le flux principal depuis la planification d'une séance jusqu'à l'évaluation.

```mermaid
sequenceDiagram
    actor C as Chef Dept
    actor T as Enseignant
    actor S as Étudiant
    participant Sess as Session Model
    participant Att as Attendance Model
    participant Grd as Grade Model
    participant Cmp as Complaint Model

    Note over C, Sess: Phase de Planification
    C->>Sess: Créer une séance (Cours, Enseignant, Salle, Classe)
    Sess-->>S: Session planifiée (visible sur l'emploi du temps)

    Note over T, Att: Phase de Cours
    T->>Att: Marquer la présence des étudiants lors de la séance
    Att-->>S: Absence/Présence notifiée

    Note over T, Grd: Phase d'Évaluation
    T->>Grd: Saisir les notes d'examen
    Grd-->>S: Note publiée

    Note over S, Cmp: Phase de Réclamation
    S->>Cmp: Déposer une réclamation sur une note
    Cmp->>C: Réclamation à traiter
    C->>Cmp: Résoudre la réclamation (Accepter/Rejeter)
    Cmp-->>S: Décision notifiée
```

---

## 3. Diagramme de Classes Global

Ce diagramme présente l'ensemble des entités du système et leurs relations de dépendance.

```mermaid
classDiagram
    class User {
        +ObjectId _id
        +String firstName
        +String lastName
        +String email
        +String role
        +ObjectId classId
        +String department
        +Boolean isActive
    }

    class Department {
        +ObjectId _id
        +String name
        +String headEmail
        +TeacherSchema[] teachers
        +ClassSchema[] classes
    }

    class Course {
        +ObjectId _id
        +String name
        +String code
        +Number semester
        +ObjectId department
    }

    class Session {
        +ObjectId _id
        +ObjectId course
        +Object teacher
        +Object room
        +Mixed classId
        +Number dayOfWeek
        +Number timeSlot
    }

    class Room {
        +ObjectId _id
        +String name
        +String building
        +Number capacity
    }

    class Attendance {
        +ObjectId _id
        +ObjectId student
        +ObjectId teacher
        +String status
        +Date date
    }

    class Grade {
        +ObjectId _id
        +ObjectId student
        +ObjectId teacher
        +Number score
        +String type
        +String semester
    }

    class Complaint {
        +ObjectId _id
        +ObjectId student
        +ObjectId grade
        +String status
        +String reason
        +String response
    }

    Department "1" -- "*" Course : contient
    Department "1" -- "*" User : regroupe (Etudiants/Profs)
    Course "1" -- "*" Session : instancie
    Room "1" -- "*" Session : héberge
    Session "1" -- "*" Attendance : génère enregistrements
    User "1" -- "*" Attendance : assiduité (Etudiant)
    User "1" -- "*" Grade : reçoit (Etudiant)
    User "1" -- "*" Grade : attribue (Enseignant)
    Grade "1" -- "0..1" Complaint : fait l'objet de
    User "1" -- "*" Complaint : dépose (Etudiant)
```
