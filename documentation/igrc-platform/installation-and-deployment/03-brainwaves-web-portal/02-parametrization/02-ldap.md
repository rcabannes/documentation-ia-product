---
title: "LDAP Access"
description: "LDAP Access"
---

# LDAP Access

Identity Analytics's Web Portal relies on the security of the underlying web application server. Tomcat is exposed to end users but relies on one or multiple LDAP repositories for authentication and authorization.

This guide explains how to use an **LDAP Directory** for user authentication and authorization by using the following features:

- **AuthN**: LDAP Directory authentication (login + password)  
- **AuthZ**: Role retrieval (direct LDAP groups considered as user roles)


## Architecture

![LDAP access architecture](./images/LDAP_access_architecture.png)

Tomcat uses a **Realm** to process authentication with the LDAP Directory via LDAP requests. The standard Tomcat feature for this is called **JNDIRealm**.

- [Tomcat 9.0 JNDIRealm Documentation](https://tomcat.apache.org/tomcat-9.0-doc/realm-howto.html#JNDIRealm)
- [Tomcat 11.0 JNDIRealm Documentation](https://tomcat.apache.org/tomcat-11.0-doc/realm-howto.html#JNDIRealm)

This architecture works on Windows and Linux with:

- Tomcat 9.0, lines **4.x** (recommended) and **3.x** (not recommended)
- Tomcat 11.0, line **5.x** (recommended)


## Prerequisites

These libraries are not available for direct download. Contact Professional Services or Support to obtain the files for your Tomcat and Java versions.

`RoleMappingRealm` (`com.brainwave.tomcat.realm.RoleMappingRealm`) is the Identity Analytics library used with `JNDIRealm`.

**4.x (recommended)** — Tomcat 9.0. These modules run on Java 8, and on Java 17.

- `tomcat-addons-realm-role-mapping-4.x.x.jar`
- `tomcat-addons-realm-base-4.x.x.jar`
- `tomcat-addons-common-4.x.x.jar`

**5.x (recommended)** — Tomcat 11.0 (Jakarta). Java 21 or later. Same JARs, with the `5.x.x` version:

- `tomcat-addons-realm-role-mapping-5.x.x.jar`
- `tomcat-addons-realm-base-5.x.x.jar`
- `tomcat-addons-common-5.x.x.jar`

**3.x (not recommended)** — Tomcat 9.0. One JAR contains every add-on:

- Java 8: `bw-tomcat-9.0-addons.java1.8-3.x.x.jar`
- Java 17 or later: `bw-tomcat-9.0-addons.java17-3.x.x.jar`

**Assumptions:**
- Tomcat is installed and available
- Operator has **read/write privileges** on necessary files and folders to proceed with installation
- Server hosting Tomcat can access LDAP/LDAPS ports (389 / 636)


## Configuration variables

The following variables will be used for configuration:

|         Variable         |                                                 Description                                                 |          Example value          |
| :----------------------: | :---------------------------------------------------------------------------------------------------------: | :-----------------------------: |
| `TOMCAT_INSTALL_FOLDER`  |                                       Tomcat installation root folder                                       |          /etc/tomcat9/          |
|   `TOMCAT_CONF_FOLDER`   |                               Folder that contains Tomcat configuration files                               |        /etc/tomcat9/conf        |
| `TOMCAT_LIB_FOLDER_HOME` |                            Folder that contains all the libraries used by the Tomcat                            |     /usr/share/tomcat9/lib      |
|      `LDAP_BW_LIB`       | Identity Analytics JARs for `RoleMappingRealm`. See the prerequisites for the file names. | tomcat-addons-realm-role-mapping-4.x.x.jar, tomcat-addons-realm-base-4.x.x.jar, tomcat-addons-common-4.x.x.jar |
| `LDAP_ROLE_MAPPING_FILE` |                        File that contains mapping between AD groups and Portal roles                        |     rolemapping.properties      |

### 1. Retrieve LDAP information 

For each LDAP directory (used for **AuthN**), retrieve the following information:

| Property                   | Description                                             | Example                      |
|-----------------------------|---------------------------------------------------------|------------------------------|
| `LDAP name`                | Name of the LDAP directory                        | ACME                         |
| `LDAP URL`                 | LDAP directory URL to be queried                                   | `ldaps://acme.com:636`       |
| `Service account`          |  LDAP service account used to query the LDAP directory. It can be either the `distiguishedName` or the `userPrincipalName`.                                 | `acme\svc-brw-prod`          |
| `Service account password` | Password of the service account used to query the LDAP directory.                              | `***`                        |
| `User base`                | Base entry that has subtree containing users                    | `DC=acme,DC=com`             |
| `User search`              | Expression that defines the criteria for matching user entries.        | `(samAccountName={0})`       |
| `User sub tree`            | The user search scope. Set to true to search the entire subtree starting from the User base entry. The default is false, which limits the search to only the top-level entries.    | `true`                       |
| `Role base`                | The base entry for the role search. If not defined, the top-level directory context is used as the default search base                            | `DC=acme,DC=com`             |
| `Role name`                | Attribute containing identifier of the role                            | `cn`                         |
| `Role sub tree`            |The role search scope. Set to true to search the entire subtree starting from the Role base entry. By default, it is false, limiting the search to only the top-level entries.                                     | `true`                       |
| `Role nested`              | Enable nested roles. Set to true to allow roles to be nested within other roles. When enabled, each discovered Role name and distinguishedName is recursively searched for additional roles. The default is false.                           | `true`                       |
| `Role search`              | The LDAP search filter used to select role entries. It can optionally include pattern replacements: {0} for the distinguished name, {1} for the username, and {2} for an attribute from the authenticated user’s directory entry.        | `(member={0})`               |


For each LDAP directory, if the client chooses to rely on the AuthZ mechanism, you will need to retrieve the following details (example values shown below):

* LDAP name: The name of the LDAP directory. This value is only for identification and does not affect technical files.

* Identity Analytics portal role: The role assigned within the Identity Analytics application. The four roles shown in the earlier table are the standard IAP roles.

* LDAP group: The name of the LDAP group that assigns the corresponding Identity Analytics role to users (used if role authorization is delegated to LDAP). This value must be the LDAP group attribute returned by the LDAP query, which is chosen in the previous AuthN table under the Role name column (e.g., the cn in our example).



| LDAP name | Identity Analytics portal role | LDAP group      |
| :-------- | :----------------------------- | :-------------- |
| ACME      | user                           | brw-users       |
| ACME      | auditor                        | brw-auditors    |
| ACME      | technical admin                | brw-tech-admins |
| ACME      | functional admin               | brw-func-admins |
| ...       | ...                            | ...             |


### 2. Configure Tomcat


> If you are using Linux, ensure you have an understanding of files and folders rights.

The second step is to configure the LDAP AuthN/AuthZ under the Tomcat:

- Under the `<TOMCAT_LIB_FOLDER_HOME>` folder, add the `<LDAP_BW_LIB>` library

- Open the `<TOMCAT_CONF_FOLDER>/server.xml` file
  - If you find a **UserDatabaseRealm** defined within the **LockOutRealm** section, comment it out.

Next, depending on the number of LDAP realms you want to configure, follow one of the two the configuration steps (single LDAP or multiple LDAP) listed below:

#### Configuration for a single LDAP

Within the <TOMCAT_CONF_FOLDER>/server.xml file, define a JNDI Realm and a RoleMapping Realm to configure the LDAP directory connection. 

```xml
<!-- Use the LockOutRealm to prevent attempts to guess user passwords via a brute-force attack -->
<Realm className="org.apache.catalina.realm.LockOutRealm">
  ...
  <Realm className="com.brainwave.tomcat.realm.RoleMappingRealm" reloadPeriod="0" configurationFile="<LDAP_ROLE_MAPPING_FILE>">
    <!-- This is the Realm to which role mapping is applied. -->
    <Realm
    className="org.apache.catalina.realm.JNDIRealm"
    connectionURL="<LDAP URL>"
    connectionName="<Technical account>"
    connectionPassword="<Technical account password>"
    authentication="simple"
    referrals="follow"
    adCompat="true"

    userBase="<User base>"
    userSearch="<User search>"

    userSubtree="<User sub tree>"
    roleBase="<Role base>"
    roleName="<Role name>"
    roleSubtree="<Role sub tree>"
    roleNested="<Role nested>"
    roleSearch="<Role search>"
    />
  </Realm>
  ...
<!-- End of LockOutRealm section -->
</Realm>
```

> All configuration values should already have been retrieved during the `LDAP information retrieving` section.  

> The **JNDIRealm** must be included within the **RoleMappingRealm**, which in turn must be included within the **LockoutRealm**.

`reloadPeriod` is a number of seconds. `0` reads the mapping file when Tomcat starts and does not read it again until the next restart. A positive value checks the file every that many seconds and reloads it when the content has changed. `reloadPeriod="10"` checks every 10 seconds.

After completing the realm configurations, the next step is to create the role mapping file:

- In the `<TOMCAT_CONF_FOLDER>` directory (`$CATALINA_BASE/conf`):
  - Create a file named `<LDAP_ROLE_MAPPING_FILE>`
  - Populate it based on your context, using the format shown below:

    ```properties
    <PORTAL_ROLE1>=<LDAP_GROUP1>
    <PORTAL_ROLE2>=<LDAP_GROUP2>
    ...
    <PORTAL_ROLEn>=<LDAP_GROUPn>
    ```

    The key is the Identity Analytics portal role. The value is the LDAP group returned by the role search (`cn` in the example below). Separate several groups for the same portal role with a comma: `user=BW_Users,BW_Contractors`.

    A value accepts three wildcards:

    - `*` as a whole value grants that portal role to every authenticated user: `user=*`. It has that meaning only when the value is exactly `*`. A group name that contains `*` is compared as that exact name.
    - `%` matches any sequence of characters. `BW%` matches a name that starts with `BW`, `%Admin` a name that ends with `Admin`, and `%Contract%` a name that contains `Contract`.
    - `?` matches one character. `A1??` matches `A100` and `AB101`.

    Wildcards can be mixed with exact group names: `user=BW_Users,BW%`.

From **3.4.0**, **4.1.0** and **5.1.0**, a bare `configurationFile` is read from `$CATALINA_BASE/conf`. An absolute path is used as given. Any other relative path is resolved from `$CATALINA_BASE` and is not prefixed with `conf/` a second time. A value that contains a directory, such as `subdir/roles.properties`, is `$CATALINA_BASE/subdir/roles.properties`.

> Identity Analytics requires at least the user-specific role to be mapped. For details on other default IAP portal roles, see the [Web Portal Roles](https://developer.radiantlogic.com/ia/iap-2.0/identity-analytics/integration-guide/03-webportal-roles/) article.

#### Configuration for multiple LDAPs

In <TOMCAT_CONF_FOLDER>/server.xml, define as many JNDI and RoleMapping Realms as needed to establish connections with all LDAP directories.

```xml
<!-- Use the LockOutRealm to prevent attempts to guess user passwords via a brute-force attack -->
<Realm className="org.apache.catalina.realm.LockOutRealm">
  ...
  <Realm className="org.apache.catalina.realm.CombinedRealm">
    <!-- First LDAP directory -->
    <Realm className="com.brainwave.tomcat.realm.RoleMappingRealm" reloadPeriod="0" configurationFile="<LDAP_ROLE_MAPPING_FILE_1>">
      <Realm
        className="org.apache.catalina.realm.JNDIRealm"
        connectionURL="<LDAP URL>"
        connectionName="<Technical account>"
        connectionPassword="<Technical account password>"
        authentication="simple"
        referrals="follow"
        adCompat="true"

        userBase="<User base>"
        userSearch="<User search>"

        userSubtree="<User sub tree>"
        roleBase="<Role base>"
        roleName="<Role name>"
        roleSubtree="<Role sub tree>"
        roleNested="<Role nested>"
        roleSearch="<Role search>"
      />
    </Realm>
    <!-- Second LDAP directory -->
    <Realm className="com.brainwave.tomcat.realm.RoleMappingRealm" reloadPeriod="0" configurationFile="<LDAP_ROLE_MAPPING_FILE_2>">
      <Realm
        className="org.apache.catalina.realm.JNDIRealm"
        connectionURL="<LDAP URL>"
        connectionName="<Technical account>"
        connectionPassword="<Technical account password>"
        authentication="simple"
        referrals="follow"
        adCompat="true"

        userBase="<User base>"
        userSearch="<User search>"

        userSubtree="<User sub tree>"
        roleBase="<Role base>"
        roleName="<Role name>"
        roleSubtree="<Role sub tree>"
        roleNested="<Role nested>"
        roleSearch="<Role search>"
      />
    </Realm>
    ...
    <!-- Nth LDAP directory -->
    <Realm className="com.brainwave.tomcat.realm.RoleMappingRealm" reloadPeriod="0" configurationFile="<LDAP_ROLE_MAPPING_FILE_N>">
      <Realm
        className="org.apache.catalina.realm.JNDIRealm"
        ...
      />
    </Realm>
  </Realm>
  ...
<!-- End of LockOutRealm section -->
</Realm>
```

> Each **JNDIRealm** must be included within its own **RoleMappingRealm**. Those **RoleMappingRealm** elements must be included within the **CombinedRealm**, which in turn must be included within the **LockOutRealm**.

After completing the realm configurations, the next step is to create the role mapping file. Use the same location rule as the single LDAP mapping file: `$CATALINA_BASE/conf` for a bare name.

- In the `<TOMCAT_CONF_FOLDER>` directory (`$CATALINA_BASE/conf`):
  - Create a file named `<LDAP_ROLE_MAPPING_FILE>`
  - Populate it based on your context, using the format shown below:

    ```properties
    <PORTAL_ROLE1>=<LDAP_GROUP1>
    <PORTAL_ROLE2>=<LDAP_GROUP2>
    ...
    <PORTAL_ROLEn>=<LDAP_GROUPn>
    ```

    The key is the Identity Analytics portal role. The value is the LDAP group. Separate several groups for the same portal role with a comma. The wildcards `*`, `%` and `?` are the same as for a single LDAP. `reloadPeriod` is a number of seconds, as for a single LDAP.

You can create as many mapping files as required by your realms.

> Identity Analytics requires at least the user-specific role to be mapped. For details on other default IAP portal roles, see the [Web Portal Roles](https://developer.radiantlogic.com/ia/iap-2.0/identity-analytics/integration-guide/03-webportal-roles/) article.

### 3. Debug settings

To debug the authentication settings, append the following lines to the end of your `<TOMCAT_CONF_FOLDER>/logging.properties`:

```properties
# Authentication Realm debug
org.apache.catalina.realm.level = ALL
org.apache.catalina.realm.useParentHandlers=true
org.apache.catalina.authenticator.level=ALL
org.apache.catalina.authenticator.useParentHandlers=true
com.brainwave.tomcat.realm.RoleMappingRealm.level = ALL
com.brainwave.tomcat.realm.RoleMappingRealm.useParentHandlers=true
```

## Examples

### JNDI and RoleMapping realms for one LDAP

```xml
<Realm className="com.brainwave.tomcat.realm.RoleMappingRealm" reloadPeriod="10" configurationFile="mappings_sample_ad_group.properties">
  <Realm
    className="org.apache.catalina.realm.JNDIRealm"
    connectionURL="ldaps://192.168.217.136:636"
    connectionName="tomcat-jndi@demo.local"
    connectionPassword="Password01"
    authentication="simple"
    referrals="follow"
    adCompat="true"

    userBase="dc=demo,dc=local"
    userSearch="(samAccountName={0})"
    userSubtree="true"

    roleBase="dc=demo,dc=local"
    roleName="cn"
    roleSubtree="true"
    roleNested="true"
    roleSearch="(member={0})"
  />
</Realm>
```

### Role mapping

```properties
user = BW_Users,BW%
technicaladmin = BW_Managers
auditor = *
```

`BW%` matches every LDAP group whose name starts with `BW`. `auditor=*` grants the `auditor` portal role to every authenticated user. `reloadPeriod="10"` on the realm above checks this file every 10 seconds.

