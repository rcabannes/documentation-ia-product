---
title: "SAML Authenticators properties"
description: "SAML Authenticators properties"
---

# SAML Authenticators properties

This page describes each property of `SAMLAuthenticator.properties`. The installation procedure, the library names, and a copy of each configuration file are on the [SAML Access](./index.md) page.

From **3.4.0**, **4.1.0** and **5.1.0**, put these files in `$CATALINA_BASE/conf`. That is the general rule. A bare file name (`SAMLAuthenticator.properties`, `idp-metadata.xml`, `sp-metadata.xml`, `rolemapping.properties`, `samlkeystore.p12`) is read from that directory. An absolute path is used as given. Any other relative path is resolved from `$CATALINA_BASE`, not from the process working directory and not from `CATALINA_HOME` when that directory is different.

## idpMetadata

Pointer to the Identity Provider metadata file. Put it in `$CATALINA_BASE/conf`. Use the file provided by the Identity Provider. An sample is in [Configuration file samples](./index.md#configuration-file-samples). An absolute path, or a path relative to `$CATALINA_BASE`, also works.

## spMetadata

Pointer to the Service Provider metadata file. Put it in `$CATALINA_BASE/conf`. The default name is `sp-metadata.xml`. The complete file is in [Configuration file samples](./index.md#configuration-file-samples). An absolute path, or a path relative to `$CATALINA_BASE`, also works.

`entityID` (sometimes called **Relying Party**) is the name of **your** application that must be registered with the Identity Provider.

The descriptor in that sample sets assertion signatures as required:

```xml
<md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="true" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
```

Update the `Location` values to match the protocol, port, and web application name. For instance `http://localhost:9090/sandbox/acs` becomes `https://localhost:8443/MyWebApp/acs`. Keep the `/acs` and `/logout` suffixes.

On 4.x and 5.x, a response is accepted when the response is signed or when every assertion is signed. When `WantAssertionsSigned` is `true`, assertions must still be signed even if the response is signed. SHA-1 is rejected on those lines.

## defaultURI

The default `URI attribute` is not mandatory but is strongly encouraged. It must contains a relative `URI` to the main portal page. In the **Identity Analytics** context it is a `/portal` URI.  
When the authentication is **Idp** initiated (the user directly connects to **Okta** and clicks on the **Identity Analytics** icon in the **Okta** portal), the user will be redirected to this `URL` once authenticated.  

A value could be: `/[your application]/portal`.  

> Note that if the authentication is **Sp** initiated, this `URL` will be **ignored**: the user will be redirected to the initially requested `URL` once authenticated.  

## forceURI

The `force URI` attribute is not mandatory. It contains a `relative URI` to the main portal page. In the **Identity Analytics** context it is a `/portal` URI.  
When the authentication is **SP** initiated the user is redirected to this `URL` once authenticated.  

A value could be: `/[your application]/portal`.  

## checkRealm

This attribute accept a boolean value (`0`/`1` or `false`/`true`). If set to `true`, once authenticated through `SAML`, the user will be pushed to the configured `Realm` with a blank password.  
It can be used to declare the **roles locally** while maintaining a delegated authentication through an **Idp**.  

If set to `true`, and if you are using the default `Realm`, you can edit the users roles in the `/conf/tomcat-users.xml` file:  

```xml
<tomcat-users xmlns="http://tomcat.apache.org/xml"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
version="1.0">
    <role rolename="tomcat"/>
    <role rolename="user"/>
    <user username="jdoe@acme.com" password="" roles="tomcat"/>
    <user username="mcarrey@acme.com" password="" roles="tomcat,user"/>
    <user username="jbegood@acme.com" password="" roles="user"/>
</tomcat-users>
```

Note that:

- The **username** value must be equal to the user value provided by the `Identity Provider`
- The **password** value must be empty  

If set to `True`, the **IdP** roles and the default roles will be **ignored**. `checkRealm`is set to **False** by default.  

## defaultRoleList

The `defaultRoleList` attribute is optional, it contains a list of roles who are associated to users who are successfully authenticated by the **identity provider**. The roles are separated by a '`,`'. The default value is `user`.  

## roleAttribute

The role attribute is optional, it contains the name of the `SAML` attribute which contains a list of role as provided back by the **Identity Provider**.  
The attribute is **empty** by default.  

## roleMapping

The role mapping attribute is **mandatory** if `roleAttribute` is set, it must point to a **property** file. The property file is used to dynamically **map the role** names provided by the **identity provider** to local roles names.  
This could be useful to map **identity provider** roles to your **Identity Analytics** application roles (such as `igrc-administrator` for instance).  

Put the file in `$CATALINA_BASE/conf`. A sample is in [Configuration file samples](./index.md#configuration-file-samples). An absolute path, or a path relative to `$CATALINA_BASE`, also works.  

This file is reloaded automatically every 10 minutes.  

## staticRole

The static role attribute is **mandatory**, it must point to a **property** file. The property file is used to declare the users roles.  
The content is in the form `login=roles` where roles is in the form `role1,role2,role3`.  

Put the file in `$CATALINA_BASE/conf`. A sample is in [Configuration file samples](./index.md#configuration-file-samples). An absolute path, or a path relative to `$CATALINA_BASE`, also works.  

This file is reloaded automatically every 10 minutes.  

## genericUsers

The generic users attribute is optional, if used it can **substitute** an individual with a **generic account**.  
It can be useful if you do not load the **identity provider** content in **Identity Analytics** and thus don't know the users who connect the portal.  

Here is how you can leverage this parameter:  

- Load some fake identities in your **Identity Analytics** ledger (for instance `standard user` and `power user`)  
- Create `standard user` and `power user` as roles in your **Idp** and attach them to your **Idp** users (only one of those roles at a time)  
- Configure the genericUsers attribute with the following value: `standard user,power user`  

We assume here that `John Doe` can connect to **Identity Analytics** and is associated with the `standard user` role:  

- `John Doe` tries to access the **Identity Analytics** instance  
- He is redirected to the **Idp**, he authenticates  
- He is redirected to the **Identity Analytics** instance  
- The `SAML` response is decoded, `John Doe` is assigned the `standard user` role  
- `standard user` is part of the `genericUsers` attribute, `John Doe` will be connected to the **Identity Analytics** instance as the `standard user` identity  

## genericUserPattern

When a generic user is found, this optional parameter can be used to format the user login. `{login}` and `{generic}` can be used and will be dynamically replaced by their corresponding value.  
For instance: if you configure this parameter with `{login} ({generic})`.  
When `jdoe@acme.com` is connecting with the generic user `user` role,  the user returned by the Principal will be:  
`jdoe@acme.com (user)`

## userAttributeIdentifier

Optional. Name of a SAML attribute used as the user login instead of the NameID. If the attribute is missing or empty, the NameID is used.

Available since **2.1**, and still available in 3.x, 4.x and 5.x.

> **Note:** Fixed in **3.3.11**, **4.0.1** and **5.0.1**. In 3.3.7 through 3.3.10, 4.0.0 and 5.0.0 the property is read only when authentication-request signature is enabled.

## privateKeystore, privateKeystorePass, privateKeyAlias, privateKeyPass, privateKeystoreFormat

Available since **3.3.7**, and in **4.x** and **5.x**. They identify the keystore used to sign authentication requests. They are required when `AuthnRequestsSigned` is true in the SP metadata, or when `WantAuthnRequestsSigned` is true in the IdP metadata. The procedure, including the `keytool` commands and the certificate to publish in `sp-metadata.xml`, is in [Sign authentication requests](./index.md#sign-authentication-requests) on the SAML Access page.

The keystore password and the private key password must be identical. `privateKeystoreFormat` is `PKCS12` for a `.p12` file. Put the keystore in `$CATALINA_BASE/conf`, or give `privateKeystore` an absolute path or a path relative to `$CATALINA_BASE`. The keystore file must be readable by the account that runs Tomcat and must not be readable by other accounts. On Linux, mode `600` or `640`. On Windows, an ACL limited to the Tomcat service account.

The examples use the alias `privatekey`. `privateKeyAlias` must use that same value. `keytool -genkeypair` stores a self-signed certificate with the private key. Keep that certificate in the keystore entry.

> **Note:** Fixed in **4.0.2**, **4.1.0**, **5.0.2** and **5.1.0**. In 4.0.0, 4.0.1, 5.0.0 and 5.0.1, a signed authentication request failed before the IdP was called (HTTP 500 on HTTP-POST, HTTP 403 on HTTP-Redirect) because the signing credential had no public key. The fixed versions sign with the X.509 certificate stored in the keystore next to the private key. An entry that contains only a private key cannot sign. Unsigned configurations are unchanged. The 3.x line is not affected.

## sigAlgo

Available since **3.3.8**. Short name of the algorithm used to sign authentication requests, for example `rsa-sha256`. Do not set a signature URI.

Supported values on 3.x: `rsa-sha1`, `rsa-sha224`, `rsa-sha256`, `rsa-sha384`, `rsa-sha512`, `dsa-sha1`, `dsa-sha256`, `ecdsa-sha1`, `ecdsa-sha224`, `ecdsa-sha256`, `ecdsa-sha384`, `ecdsa-sha512`.

On 3.3.8 through 3.3.10, the default when the property is unset is `rsa-sha1`. On **4.x** and **5.x**, the default is `rsa-sha256`. SHA-1 (`rsa-sha1`, `dsa-sha1`, `ecdsa-sha1`) is rejected on those lines, both for the authentication request and for the response and the assertions.

## One file per web application

`SAMLAuthenticator.properties` applies to every web application. Since **3.3.10**, and on **4.x** and **5.x**, create `SAMLAuthenticator-<context-name>.properties` to give one web application its own configuration, including its own `sp-metadata.xml`. When that file is absent, `SAMLAuthenticator.properties` is used.
