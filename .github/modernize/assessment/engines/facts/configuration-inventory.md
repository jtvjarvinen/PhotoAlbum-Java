# Configuration & Externalized Settings Inventory

This project has a small but clear configuration surface: Spring Boot properties, Docker Compose environment injection, and a local `.env`-based secret workflow. No external config server, vault, or feature-flag service was found.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Base application config | Spring properties | `src/main/resources/application.properties` | Default runtime settings for the app: Oracle datasource, JPA, upload limits, logging, and server encoding/port. |
| Docker profile config | Spring properties | `src/main/resources/application-docker.properties` | Overrides for containerized runs; activated by `SPRING_PROFILES_ACTIVE=docker`. |
| Test profile config | Spring properties | `src/test/resources/application-test.properties` | H2-backed test settings and test-only credentials; activated by `@ActiveProfiles("test")`. |
| Container orchestration | Docker Compose | `docker-compose.yml` | Injects runtime environment variables, sets profile selection, and defines Oracle/app startup dependency. |
| App container build/runtime | Dockerfile | `Dockerfile` | Defines build image, runtime image, exposed port, and JVM heap options. |
| Local secrets template | Env file template | `.env.example` | Template for required local secrets and credentials; actual `.env` is git-ignored. |
| Oracle bootstrap SQL | SQL init script | `oracle-init/01-create-user.sql` | Runs in the Oracle container to grant the schema user least-privilege access. |
| Oracle bootstrap helper | Shell script | `oracle-init/create-user.sh` | Reads env vars and creates/grants the application user after Oracle is ready. |
| Security configuration | Java config | `src/main/java/com/photoalbum/config/SecurityConfig.java` | Reads `app.admin.username` and `app.admin.password` from configuration. |
| Upload validation config consumer | Java service | `src/main/java/com/photoalbum/service/impl/PhotoServiceImpl.java` | Reads `app.file-upload.*` values from configuration. |
| Secret exclusion rules | Git ignore | `.gitignore` | Prevents committed `.env`/`.env.*` files except `.env.example`. |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Default Maven build (no explicit profiles) | Automatic | Compiles and packages the executable JAR. | `spring-boot-starter-parent` 2.7.18, `spring-boot-maven-plugin`. |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | Implicit when no profile is set | `src/main/resources/application.properties` | Oracle datasource defaults, `server.port=8080`, upload validation, DEBUG logging for app/web. |
| docker | `SPRING_PROFILES_ACTIVE=docker` in `docker-compose.yml` | `src/main/resources/application.properties`, `src/main/resources/application-docker.properties` | Oracle datasource values from env, Oracle dialect, INFO/WARN logging, Docker-oriented JPA settings. |
| test | `@ActiveProfiles("test")` in `src/test/java/com/photoalbum/PhotoAlbumApplicationTests.java` | `src/test/resources/application-test.properties` | H2 datasource, `create-drop`, test upload path, test admin credentials, reduced SQL logging. |

## Properties Inventory

### Main application runtime

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.charset` | `UTF-8` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.enabled` | `true` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.force` | `true` | default, docker | `application.properties`, `application-docker.properties` |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.datasource.username` | `photoalbum` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.datasource.password` | required from env (`SPRING_DATASOURCE_PASSWORD`) | default, docker; blank in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.jpa.hibernate.ddl-auto` | `create` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.jpa.show-sql` | `true` | default, docker; overridden in test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default, docker | `application.properties`, `application-docker.properties` |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | `application.properties`, `application-docker.properties` |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | `application.properties`, `application-docker.properties` |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker, test | `application.properties`, `application-docker.properties`, `application-test.properties`; consumed by `PhotoServiceImpl` |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker, test | `application.properties`, `application-docker.properties`, `application-test.properties`; consumed by `PhotoServiceImpl` |
| `app.file-upload.max-files-per-upload` | `10` | default, docker, test | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `logging.level.com.photoalbum` | `DEBUG` | default, test; `INFO` in docker | `application.properties`, `application-docker.properties`, `application-test.properties` |
| `logging.level.org.springframework.web` | `DEBUG` | default; `WARN` in docker | `application.properties`, `application-docker.properties` |
| `logging.level.org.hibernate.SQL` | not set in default; `DEBUG` in docker | docker | `application-docker.properties` |

### Security and auth-related settings

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `app.admin.username` | `admin` | default fallback in code; overridden in docker and test | `SecurityConfig.java`, `docker-compose.yml`, `.env.example`, `application-test.properties` |
| `app.admin.password` | required from config | docker and test set explicit values; no default in code | `SecurityConfig.java`, `docker-compose.yml`, `.env.example`, `application-test.properties` |

### Test-only settings

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.datasource.url` | `jdbc:h2:mem:testdb` | test | `application-test.properties` |
| `spring.datasource.username` | `sa` | test | `application-test.properties` |
| `spring.datasource.password` | empty | test | `application-test.properties` |
| `spring.datasource.driver-class-name` | `org.h2.Driver` | test | `application-test.properties` |
| `spring.jpa.database-platform` | `org.hibernate.dialect.H2Dialect` | test | `application-test.properties` |
| `spring.jpa.hibernate.ddl-auto` | `create-drop` | test | `application-test.properties` |
| `spring.jpa.show-sql` | `false` | test | `application-test.properties` |
| `app.file-upload.upload-path` | `target/test-uploads` | test | `application-test.properties` |
| `app.admin.username` | `admin` | test | `application-test.properties` |
| `app.admin.password` | `test-admin-password` | test | `application-test.properties` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory / CPU | Instance Count |
|---|---|---|---|
| `photoalbum-java-app` | `SPRING_PROFILES_ACTIVE=docker`; `SPRING_DATASOURCE_URL=jdbc:oracle:thin:@oracle-db:1521/FREEPDB1`; `SPRING_DATASOURCE_USERNAME=${APP_USER:-photoalbum}`; `SPRING_DATASOURCE_PASSWORD=${APP_USER_PASSWORD}`; `APP_ADMIN_USERNAME=${APP_ADMIN_USERNAME:-admin}`; `APP_ADMIN_PASSWORD=${APP_ADMIN_PASSWORD}` | JVM heap in image: `-Xms256m -Xmx512m`; no container limit declared | 1 |
| `oracle-db` | `ORACLE_PASSWORD`, `APP_USER`, `APP_USER_PASSWORD` from Compose/env; init script mounts under `/container-entrypoint-initdb.d` | No explicit limit in Compose; README calls for at least 4GB Docker memory overall; Oracle XE/Free container has tight CPU/memory limits in practice | 1 |

## Startup Dependency Chain

1. `oracle-db` starts first and waits for Oracle to become ready, using `healthcheck.sh` with a `start_period: 180s`.
2. `oracle-init/01-create-user.sql` runs in the Oracle container initialization path to grant schema privileges.
3. `oracle-init/create-user.sh` is a helper that waits for Oracle, then creates/grants the application user from env vars.
4. `photoalbum-java-app` starts only after `oracle-db` is healthy via `depends_on: condition: service_healthy`.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ORACLE_PASSWORD` | Oracle admin password | `.env` / Compose env / Oracle init env (`[MASKED]`) |
| `APP_USER_PASSWORD` | Oracle application schema password | `.env` / Compose env / Oracle init env (`[MASKED]`) |
| `SPRING_DATASOURCE_PASSWORD` | App datasource password | Compose env, sourced from `APP_USER_PASSWORD` (`[MASKED]`) |
| `app.admin.password` / `APP_ADMIN_PASSWORD` | HTTP Basic admin password | `.env` / Compose env / `application-test.properties` (`[MASKED]`) |

### Secrets Provisioning Workflow

The local workflow is file-based: copy `.env.example` to `.env`, replace the placeholder values, and let Docker Compose inject those values into the Oracle and application containers. The Oracle container uses `ORACLE_PASSWORD`, `APP_USER`, and `APP_USER_PASSWORD` to create the schema user and grant least-privilege access; the app container receives the same schema password as `SPRING_DATASOURCE_PASSWORD` plus `APP_ADMIN_USERNAME` / `APP_ADMIN_PASSWORD` for HTTP Basic auth. No Vault, Key Vault, AWS Secrets Manager, or external secrets broker was found in this repo.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | No `@ConditionalOnProperty`, `@ConditionalOnExpression`, LaunchDarkly, Unleash, or similar flag wiring was found. |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot parent | `2.7.18` | `pom.xml` |
| Java target | `1.8` / `8` | `pom.xml` |
| Maven compiler source/target | `8` | `pom.xml` |
| `spring-boot-maven-plugin` | Inherited from Spring Boot parent | `pom.xml` |
| Spring Boot starters (`web`, `thymeleaf`, `data-jpa`, `validation`, `security`, `test`, `devtools`) | Managed by Spring Boot 2.7.18 | `pom.xml` |
| Commons IO | `2.11.0` | `pom.xml` |
| Oracle JDBC driver (`ojdbc8`) | Version not declared in `pom.xml` | `pom.xml` |
| H2 | Test scope; version managed by dependency management | `pom.xml` |
| Build image | `maven:3.9.6-eclipse-temurin-8` | `Dockerfile` |
| Runtime image | `eclipse-temurin:8-jre` | `Dockerfile` |
| Oracle container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
