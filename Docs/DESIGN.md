# Projet PFE : Documentation du Design (Mermaid UML)

Ce document présente le design du système pour chaque sprint, incluant les diagrammes de cas d'utilisation, de séquences et de classes.

---

## Sprint 1 : Gestion des utilisateurs

### 1.1 Diagramme des cas d'utilisation

```mermaid
graph TD
    Student[Étudiant]
    Teacher[Enseignant]
    Chef[Chef de Département]
    Admin[Administrateur]

    subgraph "Authentification & Profil"
        UC1(S'inscrire - Register)
        UC2(Se connecter - Login)
        UC3(Consulter son profil)
    end

    subgraph "Administration des utilisateurs"
        UC4(Lister les étudiants)
        UC5(Gérer les utilisateurs)
    end

    Student --> UC1
    Student --> UC2
    Student --> UC3

    Teacher --> Student
    Teacher --> UC4

    Chef --> Teacher
    Chef --> UC5

    Admin --> Chef
```

### 1.2 Diagramme de séquence (Authentification - Login)

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant R as Router (authRoutes)
    participant C as AuthController
    participant M as User Model (Mongoose)
    participant J as JWT Utility

    U->>R: POST /api/auth/login (email, password)
    R->>C: loginUser(req, res)
    C->>M: findOne({ email }).select('+password')
    M-->>C: user (ou null)

    alt Utilisateur existe & mot de passe correct
        C->>M: comparePassword(password)
        M-->>C: true
        C->>J: generateToken(user._id)
        J-->>C: token
        C-->>U: 200 OK (User Data + Token)
    else Identifiants invalides
        C-->>U: 401 Unauthorized (Invalid credentials)
    end
```

### 1.3 Diagramme de classe (Modèle User)

```mermaid
classDiagram
    class User {
        +String firstName
        +String lastName
        +String email
        +String password
        +String role
        +String avatar
        +Boolean isActive
        +String department
        +String studentId
        +String registrationNumber
        +ObjectId classId
        +String className
        +String teacherId
        +Date createdAt
        +Date updatedAt
        +comparePassword(candidatePassword)
    }

    note for User "Roles: ADMIN, STUDENT, TEACHER, PARTNER, CHEF_DEPT"
```

---

## Sprint 2 : Gestion des emplois du temps

### 2.1 Diagramme des cas d'utilisation

```mermaid
graph TD
    Student[Étudiant]
    Teacher[Enseignant]
    Chef[Chef de Département]

    subgraph "Consultation"
        UC1(Consulter son emploi du temps)
        UC2(Consulter les cours enseignés)
        UC3(Consulter les salles)
    end

    subgraph "Gestion Académique"
        UC4(Gérer les séances - CRUD)
        UC5(Gérer les cours - Modules)
        UC6(Affecter Enseignant/Salle)
    end

    Student --> UC1
    Teacher --> UC1
    Teacher --> UC2

    Chef --> Teacher
    Chef --> UC3
    Chef --> UC4
    Chef --> UC5
    Chef --> UC6
```

### 2.2 Diagramme de séquence (Consultation Emploi du Temps Étudiant)

```mermaid
sequenceDiagram
    participant S as Étudiant
    participant R as Router (academicRoutes)
    participant C as AcademicController
    participant D as Dept Model
    participant M as Session Model

    S->>R: GET /api/academic/schedule/student
    R->>C: getStudentSchedule(req, res)
    C->>D: findOne({ 'classes._id': req.user.classId })
    D-->>C: dept & classObj
    C->>M: find({ classId: queryId }).sort({ dayOfWeek, timeSlot })
    M-->>C: sessions[]
    C-->>S: 200 OK (sessions[])
```

### 2.3 Diagramme de classe (Gestion Académique)

```mermaid
classDiagram
    class Department {
        +String name
        +String head
        +String headEmail
        +Teacher[] teachers
        +Class[] classes
    }

    class Course {
        +String name
        +String code
        +Number semester
        +Number level
        +Object hours
        +ObjectId department
    }

    class Session {
        +ObjectId course
        +String courseName
        +Object teacher
        +Object room
        +Mixed classId
        +String className
        +String type
        +Number dayOfWeek
        +Number timeSlot
    }

    class Room {
        +String name
        +String building
        +Number capacity
        +String type
    }

    Department "1" -- "*" Course : contient
    Department "1" -- "*" Session : gère
    Course "1" -- "*" Session : instancie
    Room "1" -- "*" Session : accueille
```

---

## Sprint 3 : Gestion des absences

### 3.1 Diagramme des cas d'utilisation

```mermaid
graph TD
    Student[Étudiant]
    Teacher[Enseignant]
    Chef[Chef de Département]

    subgraph "Consultation Présence"
        UC1(Consulter ses absences)
        UC2(Consulter l'historique de présence)
    end

    subgraph "Saisie de Présence"
        UC3(Marquer la présence - Individuel)
        UC4(Marquer la présence - En masse)
        UC5(Modifier/Supprimer une présence)
    end

    Student --> UC1
    Teacher --> UC2
    Teacher --> UC3
    Teacher --> UC4
    Teacher --> UC5

    Chef --> Teacher
```

### 3.2 Diagramme de séquence (Marquage de présence en masse)

```mermaid
sequenceDiagram
    participant T as Enseignant
    participant R as Router (attendanceRoutes)
    participant C as AttendanceController
    participant M as Attendance Model

    T->>R: POST /api/attendance/bulk (courseName, date, records[])
    R->>C: markBulkAttendance(req, res)
    Note over C: Map records to Attendance docs
    C->>M: insertMany(docs)
    M-->>C: createdDocs[]
    C-->>T: 201 Created (count)
```

### 3.3 Diagramme de classe (Gestion des Absences)

```mermaid
classDiagram
    class Attendance {
        +ObjectId student
        +ObjectId teacher
        +String courseName
        +Date date
        +Number durationHours
        +String status
        +String sessionType
        +String justification
        +Boolean justified
        +Date createdAt
    }

    class User {
        <<Referenced>>
    }

    Attendance "*" -- "1" User : concerne (student)
    Attendance "*" -- "1" User : marqué par (teacher)
```

---

## Sprint 4 : Gestion des notes

### 4.1 Diagramme des cas d'utilisation

```mermaid
graph TD
    Student[Étudiant]
    Teacher[Enseignant]
    Chef[Chef de Département]

    subgraph "Consultation Notes"
        UC1(Consulter ses notes)
        UC2(Consulter les notes attribuées)
    end

    subgraph "Saisie & Import"
        UC3(Ajouter une note)
        UC4(Importer des notes - Excel)
        UC5(Supprimer une note)
    end

    subgraph "Réclamations"
        UC6(Déposer une réclamation)
        UC7(Traiter une réclamation)
    end

    Student --> UC1
    Student --> UC6

    Teacher --> UC2
    Teacher --> UC3
    Teacher --> UC4
    Teacher --> UC5

    Chef --> Teacher
    Chef --> UC7
```

### 4.2 Diagramme de séquence (Import de notes via Excel)

```mermaid
sequenceDiagram
    participant T as Enseignant
    participant R as Router (gradeRoutes)
    participant C as GradeController
    participant X as XLSX Utility
    participant U as User Model
    participant G as Grade Model

    T->>R: POST /api/grades/bulk-upload (file, meta-data)
    R->>C: bulkUploadGrades(req, res)
    C->>X: read(file.buffer)
    X-->>C: rows[]
    loop Chaque ligne
        C->>U: findOne({ studentId: sid })
        U-->>C: student
        Note over C: Valider score & infos
    end
    C->>G: insertMany(gradesToCreate)
    G-->>C: result
    C-->>T: 201 Created (results)
```

### 4.3 Diagramme de classe (Gestion des Notes & Réclamations)

```mermaid
classDiagram
    class Grade {
        +ObjectId student
        +ObjectId teacher
        +String courseName
        +String department
        +String subject
        +Number score
        +Number coefficient
        +String semester
        +String type
        +Date date
    }

    class Complaint {
        +ObjectId student
        +ObjectId grade
        +String reason
        +String status
        +String response
        +ObjectId resolvedBy
        +Date resolvedAt
    }

    class User {
        <<Referenced>>
    }

    Grade "*" -- "1" User : attribué à (student)
    Grade "*" -- "1" User : donné par (teacher)
    Complaint "*" -- "1" User : déposé par (student)
    Complaint "*" -- "1" Grade : concerne
    Complaint "*" -- "0..1" User : résolu par (Chef/Admin)
```

---
