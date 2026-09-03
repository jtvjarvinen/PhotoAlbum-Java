# Architecture Diagram

Photo Album is a Spring Boot 2.7.18 web application that uses Thymeleaf for server-rendered pages and Oracle Database for photo storage. The diagrams below show the main application layers and the key component interactions that connect the gallery, upload, detail, and security flows.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        clientBrowser["Web Browser"]
        clientScript["Upload JavaScript"]
    end

    subgraph Presentation["Presentation Layer"]
        presentationMvc["Spring MVC Controllers"]
        presentationViews["Thymeleaf Templates"]
    end

    subgraph Business["Business Layer - Spring Boot 2.7.18"]
        businessService["Photo Service"]
        businessSecurity["Spring Security"]
    end

    subgraph Data["Data Layer"]
        dataJpa["Spring Data JPA Repository"]
        dataModel["Photo Entity"]
    end

    subgraph Storage["Storage Layer"]
        storageOracle["Oracle Database Free 23ai"]
    end

    subgraph External["External Services"]
        externalBootstrap["Bootstrap CDN"]
    end

    clientBrowser -->|"requests pages"| presentationMvc
    clientScript -->|"POST upload requests"| presentationMvc
    presentationViews -->|"renders UI"| clientBrowser
    presentationMvc -->|"returns views"| presentationViews
    presentationMvc -->|"delegates requests"| businessService
    presentationMvc -->|"secured by"| businessSecurity
    businessService -->|"reads and writes photos"| dataJpa
    dataJpa -->|"maps entities"| dataModel
    dataJpa -->|"SQL and BLOB access"| storageOracle
    presentationViews -->|"styles and scripts"| externalBootstrap
    clientBrowser -->|"requests images"| presentationMvc
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---:|---|
| Presentation | Spring MVC | 5.x via Spring Boot 2.7.18 | Handles gallery, detail, and upload requests |
| Presentation | Thymeleaf | 3.x via Spring Boot 2.7.18 | Server-side HTML rendering |
| Business | Spring Boot | 2.7.18 | Application runtime and dependency management |
| Business | Spring Security | 5.x via Spring Boot 2.7.18 | Protects upload and delete endpoints |
| Data | Spring Data JPA | 2.x via Spring Boot 2.7.18 | Repository abstraction for photo persistence |
| Data | Hibernate JPA | 5.x via Spring Boot 2.7.18 | ORM and entity mapping |
| Storage | Oracle Database Free | 23ai | Stores photo metadata and BLOB image data |
| Runtime | Java | 8 | Application and container runtime |

### Data Storage & External Services

Photos are stored as BLOBs in Oracle Database Free 23ai along with metadata such as filename, MIME type, size, upload time, and dimensions. The web UI also depends on Bootstrap from a public CDN for styling and layout.

### Key Architectural Decisions

- Uses server-rendered Thymeleaf pages instead of a separate frontend SPA.
- Stores image bytes directly in Oracle rather than on the filesystem.
- Protects upload and delete operations with HTTP Basic auth and Spring Security.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        compHome["HomeController"]
        compDetail["DetailController"]
        compPhotoFile["PhotoFileController"]
        compUploadJs["upload.js"]
        compIndexView["index.html"]
        compDetailView["detail.html"]
    end

    subgraph Business["Business Logic"]
        compPhotoService["PhotoService"]
        compPhotoServiceImpl["PhotoServiceImpl"]
    end

    subgraph DataAccess["Data Access"]
        compPhotoRepository["PhotoRepository"]
        compPhotoEntity["Photo"]
        compUploadResult["UploadResult"]
    end

    subgraph Infrastructure["Infrastructure"]
        compSecurity["SecurityConfig"]
    end

    compHome -->|"loads photos"| compPhotoService
    compHome -->|"renders"| compIndexView
    compUploadJs -->|"POST /upload"| compHome
    compDetail -->|"loads photo and navigation"| compPhotoService
    compDetail -->|"renders"| compDetailView
    compPhotoFile -->|"fetches photo bytes"| compPhotoService
    compPhotoService -->|"implemented by"| compPhotoServiceImpl
    compPhotoServiceImpl -->|"queries and saves"| compPhotoRepository
    compPhotoServiceImpl -->|"returns results"| compUploadResult
    compPhotoRepository -->|"maps to"| compPhotoEntity
    compPhotoServiceImpl -->|"creates photo records"| compPhotoEntity
    compSecurity -.->|"protects POST requests"| compHome
    compSecurity -.->|"protects delete action"| compDetail
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| HomeController | Presentation | Spring MVC Controller | Serves the gallery page and handles photo uploads |
| DetailController | Presentation | Spring MVC Controller | Shows a single photo and handles deletion |
| PhotoFileController | Presentation | Spring MVC Controller | Streams photo bytes from storage to the browser |
| upload.js | Presentation | Client Script | Handles drag and drop upload interaction |
| index.html | Presentation | Thymeleaf Template | Renders the main gallery view |
| detail.html | Presentation | Thymeleaf Template | Renders the photo detail view |
| PhotoService | Business Logic | Service Interface | Defines photo operations |
| PhotoServiceImpl | Business Logic | Service Implementation | Validates uploads and coordinates persistence |
| PhotoRepository | Data Access | JPA Repository | Reads and writes photo records in Oracle |
| Photo | Data Access | JPA Entity | Stores photo metadata and BLOB content |
| UploadResult | Data Access | DTO | Carries upload success or failure details |
| SecurityConfig | Infrastructure | Security Configuration | Restricts upload and delete endpoints |
