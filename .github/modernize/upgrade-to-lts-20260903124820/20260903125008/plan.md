# Upgrade Plan: PhotoAlbum (20260903125008)

- **Generated**: 2025-09-03 12:50:08 UTC
- **HEAD Branch**: main
- **HEAD Commit ID**: de25211 (Fix CWE-798/CWE-862/CWE-400 in PhotoAlbum-Java)

## Available Tools

**JDKs**
- JDK 25.0.4.1: C:\Program Files\Microsoft\jdk-25.0.4.101-hotspot\bin (current target JDK, used throughout)
- JDK 25.0.2: C:\Users\JoonasJärvinen\AppData\Local\jdks\jdk-25.0.2\bin (alternative Java 25)

**Build Tools**
- Maven 3.9.16: C:\Users\JoonasJärvinen\.maven\maven-3.9.16\bin (available, compatible with Java 25)
- No Maven Wrapper (mvnw) found in project

## Guidelines

> Note: Automatic flow mode - no user input pauses. All defaults accepted. Complete end-to-end execution.

## Options

- Working branch: appmod/java-upgrade-20260903125008
- Run tests before and after the upgrade: true

## Upgrade Goals

- **Java**: 8 → 25
- **Spring Boot**: 2.7.18 → 4.0.x (latest 4.0)
- **Spring Framework**: 5.3.x (via Boot 2.7.18) → 7.x (via Boot 4.0)
- **Jakarta EE**: Migrate all javax.* to jakarta.* for Spring Boot 4.0 compliance

## Technology Stack

| Technology/Dependency | Current | Min Compatible | Why Incompatible |
|----------------------|---------|-----------------|-----------------|
| Java | 8 | 25 | User requested |
| Spring Boot | 2.7.18 | 4.0.0 | User requested; EOL support ends Q4 2026 |
| Spring Framework ⚠️ | 5.3.x | 7.0 | Bundled with Spring Boot 4.0; 5.x EOL |
| Maven | 3.9.16 | 3.9.16 | Fully compatible with Java 25 |
| maven-compiler-plugin ⚠️ | 3.11.0 | 3.13.0+ | Update recommended for Java 25 support |
| maven-surefire-plugin ⚠️ | 3.1.2 | 3.1.2 | Java 25 compatible; upgrade available |
| spring-boot-starter-web | 2.7.18 | 4.0.0 | Bundles Spring Framework 6.1.x → 7.x changes |
| spring-boot-starter-security | 2.7.18 | 4.0.0 | Major DSL/API changes in Spring Security 6.4+ |
| javax.persistence ⚠️ EOL | 2.2 (JPA 2.2) | N/A | Replaced by jakarta.persistence in Spring Boot 3.0+ |
| javax.validation ⚠️ EOL | 2.0 (Bean Validation 2.0) | N/A | Replaced by jakarta.validation in Spring Boot 3.0+ |
| javax.imageio | 11 (JDK module) | 25 | Built-in; namespace unchanged (not javax.servlet) |
| commons-io | 2.11.0 | 2.15.x | Update for security and Java 25 compat |
| ojdbc8 ⚠️ | version-managed | 23.x | Oracle JDBC 8 lacks Java 25 support; upgrade to 23.x required |
| H2 Database | version-managed | 2.2.x | Test database; update recommended for Java 25 |

## Derived Upgrades

**From Java 8 → 25:**
- Maven 3.9.16 (already available, compatible) ✓
- maven-compiler-plugin: 3.11.0 → 3.13.0+ (recommended for better Java 25 support)
- maven-surefire-plugin: 3.1.2 → 3.1.2+ (already Java 25 compatible)
- No Kotlin detected in project (no version constraint)
- JDK 25 language features available (records, sealed classes, pattern matching, etc.)

**From Spring Boot 2.7.18 → 4.0:**
- Spring Framework 5.3.x → 7.0+ (bundled with Spring Boot 4.0)
- **CRITICAL**: javax.* → jakarta.* migration (javax.persistence, javax.validation, javax.servlet, etc.)
- Spring Security API changes: DSL overhaul (authorizeRequests → authorizeHttpRequests; antMatchers → requestMatchers; lambda-based config)
- Remove javax.servlet references; validate jakarta.servlet used by Spring Web
- Update all Spring Boot starters to 4.0.x versions

**From Jakarta EE namespace migration:**
- javax.persistence.* → jakarta.persistence.*
- javax.validation.* → jakarta.validation.*
- javax.imageio.* → javax.imageio.* (no change; not javax.servlet)

## Impact Analysis

### Dependency Changes

| File | Dependency | Current | Action | Target | Reason |
|------|-----------|---------|--------|--------|--------|
| pom.xml | spring-boot-starter-parent | 2.7.18 | upgrade | 4.0.0 | User requested; unlocks Spring Boot 4.0 |
| pom.xml | java.version property | 1.8 | upgrade | 25 | User requested; target Java 25 |
| pom.xml | maven.compiler.source | 8 | upgrade | 25 | Target Java version |
| pom.xml | maven.compiler.target | 8 | upgrade | 25 | Target Java version |
| pom.xml | commons-io | 2.11.0 | upgrade | 2.16.1 | Security & Java 25 compat |
| pom.xml | ojdbc8 | version-managed (23.4) | replace | ojdbc11:23.4 | Java 25 support; ojdbc8 deprecated |
| pom.xml (auto via BOM) | maven-compiler-plugin | 3.11.0 | upgrade | 3.13.0 | Java 25 support (optional but recommended) |
| pom.xml (auto via BOM) | maven-surefire-plugin | 3.1.2 | keep | 3.1.2+ | Already Java 25 compatible |

### Source Code Changes

| File | Location | Current | Required Change | Reason |
|------|----------|---------|----------------|--------|
| Photo.java | line 3 | import javax.persistence.* | Replace with: import jakarta.persistence.* | Jakarta EE 10 namespace (Spring Boot 3.0+) |
| Photo.java | line 4-7 | import javax.validation.* | Replace with: import jakarta.validation.* | Jakarta EE 10 namespace (Spring Boot 3.0+) |
| PhotoServiceImpl.java | line 14-16 | import javax.imageio.* | Keep as-is | javax.imageio is JDK built-in; NOT javax.servlet |
| SecurityConfig.java | line 6-14 | Spring Security imports | Update to latest Spring Security 6.4+ imports | Spring Security API changes in Spring Boot 4.0 |
| SecurityConfig.java | line 51-60 | DSL: csrf().disable(), sessionManagement(), authorizeRequests(), antMatchers() | Rewrite to: securityFilterChain(http).csrf(csrf -> csrf.disable()).sessionManagement(...).authorizeHttpRequests(authz -> authz.requestMatchers(...).authenticated().anyRequest().permitAll()).httpBasic() | Spring Security 6.0+ removed deprecated DSL; lambda-based config required |
| All controller/service/model files | — | Spring imports (non-servlet) | Keep as-is | Non-servlet Spring imports (org.springframework.stereotype, org.springframework.beans, org.springframework.data) unchanged |

### Configuration Changes

No configuration file changes required. The following remain compatible:
- application.properties/application.yml: Spring Boot properties compatible across versions (may need spring.data.jpa.* adjustments if used)
- No web.xml present (Spring Boot auto-config)
- No XML Spring config files detected

### CI/CD Changes

| File | Location | Current | Required Change |
|------|----------|---------|----------------|
| Dockerfile | line 1 | FROM openjdk:8-jdk-slim OR similar | Change to: FROM openjdk:25-jdk-slim (or microsoft jdk-25 image) |
| Dockerfile | line N (if present) | MAVEN_VERSION or build reference | If Maven is pinned, update to 3.9.16 or 3.9.x latest |
| azure-pipelines.yml (if present) | Build step | jdkVersion: 8 OR JAVA_HOME pointing to 8 | Change to: jdkVersion: 25 or JAVA_HOME pointing to Java 25 |
| GitHub Actions workflow (if present) | setup-java step | java-version: 8 | Change to: java-version: 25 |

### Risks & Warnings

1. **Spring Security DSL Rewrite (SecurityConfig.java:45-62)**: Non-trivial rewrite. Spring Security 6.0+ removed the old lambda-less builder DSL (csrf().disable(), authorizeRequests(), antMatchers()). Required changes:
   - Replace .csrf().disable() with .csrf(csrf -> csrf.disable())
   - Replace .sessionManagement().sessionCreationPolicy(...) with .sessionManagement(sm -> sm.sessionCreationPolicy(...))
   - Replace .authorizeRequests() with .authorizeHttpRequests()
   - Replace .antMatchers(HttpMethod.POST, ...) with .requestMatchers(HttpMethod.POST, ...)
   - Ensure lambda syntax is used throughout chain
   **Mitigation**: Rewrite SecurityConfig to use latest Spring Security 6.4+ DSL; verify with existing unit tests (PhotoAlbumApplicationTests); ensure HTTP 403 responses for unauthorized requests.

2. **javax vs jakarta imports**: Must be 100% replaced. Missing any javax.* in a Spring Boot 4.0 app will cause ClassNotFoundException at runtime.
   **Mitigation**: Grep scan for all javax.* imports post-migration and verify jakarta.* is used.

3. **Oracle JDBC Driver (ojdbc8 → ojdbc11)**: Version change may affect connection pooling or driver behavior.
   **Mitigation**: Verify Oracle DB connection in integration tests; ojdbc11 is stable and widely tested with Java 25.

4. **Build Tool Compatibility**: Maven 3.9.16 is confirmed compatible with Java 25 (Maven does not gate JDK version, delegates to javac).
   **Mitigation**: Verify `mvn clean compile test-compile` and `mvn clean test` succeed post-upgrade; no additional steps required.

5. **Module System (Java 9+ feature, Java 25 has strong encapsulation)**: Project does not use JPMS (no module-info.java found). No runtime --add-opens workarounds needed unless reflection into java.base is detected.
   **Mitigation**: Compile and test; if module access errors occur, document and apply minimal --add-opens flag.

6. **commons-io 2.11.0 is legacy; 2.16.1 has critical bug fixes**:
   **Mitigation**: Upgrade to 2.16.1; no API changes expected.

## Upgrade Steps

### Step 1: Setup Environment
- **Rationale**: Verify Maven and Java 25 are available before starting code changes. This is mandatory for all subsequent steps.
- **Changes to Make**: Confirm Maven 3.9.16 and Java 25.0.4.1 are in PATH/available. No code changes.
- **Verification**: `mvn -v` shows Maven 3.9.16; `java -version` shows Java 25.0.4.1. Expected Result: Both commands succeed.

### Step 2: Setup Baseline (Optional)
- **Rationale**: Establish baseline test pass rate with current Java 8 + Spring Boot 2.7.18 before upgrades. This forms acceptance criteria (100% or ≥ baseline).
- **Changes to Make**: None; run tests only.
- **Verification**: Command: `mvn -DskipTests=false clean test -DargLine="-Xmx1024m"` with Java 8. Expected Result: All tests pass; record total count and pass rate.
- **Note**: If Java 8 is not available, skip this step with status "skipped".

### Step 3: Upgrade Spring Boot 2.7.18 → 4.0.0
- **Rationale**: This step unlocks Java 25 compatibility and Spring Framework 7.x. Intermediate version 3.x cannot be skipped due to major namespace changes (javax → jakarta) and API overhauls. Upgrading directly is feasible for this small project.
- **Changes to Make**:
  - Update pom.xml: spring-boot-starter-parent 2.7.18 → 4.0.0
  - Update pom.xml: java.version 1.8 → 25
  - Update pom.xml: maven.compiler.source/target 8 → 25
  - Update pom.xml: commons-io 2.11.0 → 2.16.1
  - Replace ojdbc8 with ojdbc11 (version 23.4 or later)
- **Verification**: Command: `mvn clean test-compile -DskipTests=true -Dmaven.test.skip=true -q` with Java 25. Expected Result: Compilation SUCCESS (main + test classes both compiled).

### Step 4: Migrate javax → jakarta Imports
- **Rationale**: Spring Boot 4.0 uses Jakarta EE 10, which renames all javax.* to jakarta.*.
- **Changes to Make**:
  - Photo.java: Replace import javax.persistence.* with jakarta.persistence.*
  - Photo.java: Replace import javax.validation.* with jakarta.validation.*
  - All @Entity, @Column, @NotBlank, @Size, @NotNull, @Positive annotations now use jakarta.* namespaces
  - javax.imageio remains unchanged (JDK built-in, not servlet-related)
  - Verify no javax.servlet imports are used (not found in current codebase; Spring provides jakarta.servlet)
- **Verification**: Command: `mvn clean test-compile -DskipTests=true -Dmaven.test.skip=true -q`. Expected Result: Compilation SUCCESS. Grep confirm: No "import javax.persistence\|import javax.validation" except javax.imageio.

### Step 5: Rewrite Spring Security Config
- **Rationale**: Spring Security 6.0+ removed the lambda-less builder API. SecurityConfig.java must be rewritten to use lambda-based DSL.
- **Changes to Make**:
  - SecurityConfig.java: Replace entire securityFilterChain() method with lambda-based Spring Security 6.4+ DSL:
    - `.csrf().disable()` → `.csrf(csrf -> csrf.disable())`
    - `.sessionManagement().sessionCreationPolicy(...)` → `.sessionManagement(sm -> sm.sessionCreationPolicy(...))`
    - `.authorizeRequests()` → `.authorizeHttpRequests()`
    - `.antMatchers(HttpMethod.POST, "/upload", "/detail/*/delete").authenticated()` → `.requestMatchers(HttpMethod.POST, "/upload", "/detail/*/delete").authenticated()`
    - `.anyRequest().permitAll()` → `.anyRequest().permitAll()` (unchanged)
    - `.and()` chains removed; use inline lambdas
  - Update imports: ensure all Spring Security 6.4+ classes are imported
- **Verification**: Command: `mvn clean test-compile -DskipTests=true -Dmaven.test.skip=true -q`. Expected Result: Compilation SUCCESS.

### Step 6: Final Validation
- **Rationale**: Verify all goals are met: compilation passes, all tests pass (100% or ≥ baseline), and no TODOs/workarounds remain.
- **Changes to Make**: None; test only. Fix any remaining test failures (iterative loop until 100% pass or baseline met).
- **Verification**: 
  - Command: `mvn clean test-compile -q` → Expected Result: SUCCESS
  - Command: `mvn clean test -q` → Expected Result: All tests pass (or ≥ baseline pass rate)
  - Grep: Confirm no javax.persistence, javax.validation imports remain (javax.imageio is OK)
  - Review: Ensure SecurityConfig uses lambda DSL throughout

### Step 7: CVE Validation & Fix (Post-Upgrade)
- **Rationale**: Scan for CVEs in final dependency set and fix any high-severity issues.
- **Changes to Make**: Extract direct dependencies, scan with `#validate-cves-for-java`, upgrade CVE-affected versions if patches available.
- **Verification**: Re-scan after fixes to confirm resolution.

---

## Notes

- **Automatic Flow**: This upgrade executes without user input pauses. All decisions are pre-determined in this plan.
- **Build Tool**: Maven 3.9.16 available; no wrapper in project. All mvn commands explicitly reference this installation.
- **Testing**: Baseline test pass rate will be captured in Step 2 (if Java 8 available) and used as acceptance criteria in Step 6.
- **Deferred Work**: None anticipated. Spring Security rewrite is non-trivial but fully specified in Step 5.
- **Known Issues**: Spring Security DSL rewrite is the primary complexity; mitigation is provided in Risks & Warnings.
