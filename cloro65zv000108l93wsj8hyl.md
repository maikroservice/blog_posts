---
title: "Step-By-Step Guide to setup Identity Management for wazuh with authentik"
datePublished: Thu Nov 09 2023 20:56:47 GMT+0000 (Coordinated Universal Time)
cuid: cloro65zv000108l93wsj8hyl
slug: identity-management-wazuh-and-authentik
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699526068023/0fd774fa-208c-4052-ba5c-ef0091a6cc16.png
tags: hacking, cybersecurity-1, blueteam, siem

---

Centralized Identity Management when done right gives you and your users the freedom to only remember one password to login to multiple services - single sign-on coupled with Multi-Factor Authentication (MFA) increases security - but... how would you start?

# Getting started with Identity Management

The first step could be to define a use-case in your IT environment (home lab or business) where you would want/need a single-sign on solution.

Let us assume you have a Security Information and Event Management System (SIEM) - and let us assume that your favorite system is wazuh (https://wazuh.com). Wazuh has it's own user database/management, and lucky for us wazuh supports external identity providers as well.

*Ok, but what does an identity provider to actually?*

## Identity Provider Overview

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699530128915/d744292a-aa08-4650-88e9-83e53fb9ffd4.png align="center")

Essentially, an identity provider is a piece of software that sits in the middle between your users and the service they want to use.

It takes username/password and sometimes another factor (one-time passcode, fingerprint/biometric data, key-fobs) and when the correct ones were provided it sends a token (long text) to the connected services.

These tokens allow other connected services to know WHO the user is and IF they are authenticated correctly.

## authentik container setup

The easiest way to setup authentik is to use docker compose ([https://goauthentik.io/docs/installation/docker-compose](https://goauthentik.io/docs/installation/docker-compose)) - first you download the `docker-compose.yml`, then generate secure passwords in an environment file, define the outgoing ports and spin up the containers.

```bash
mkdir authentik && cd authentik

wget https://goauthentik.io/docker-compose.yml

wget https://get.docker.com/ -O get-docker.sh

bash get-docker.sh

# optionally - install pwgen to generate passwords
sudo apt-get install -y pwgen

echo "PG_PASS=$(pwgen -s 40 1)" >> .env
echo "AUTHENTIK_SECRET_KEY=$(pwgen -s 50 1)" >> .env
echo "COMPOSE_PORT_HTTP=80" >> .env
echo "COMPOSE_PORT_HTTPS=443" >> .env 

docker compose up -d
```

You can run `docker ps` to check if all the containers are running correctly, should look like the screenshot below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533996003/f249b2ec-e165-4293-b5c6-99bdab8c8cef.png align="center")

### Initial authentik admin account setup

The next step is to create a password for the default administrator account (`akadamin)` for authentik by visiting:

`https://<IP_or_hostname_of_authentik>/if/flow/initial-setup/`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699532565441/d8f56fe8-1fd9-4ca6-97c5-1b9560af3ab6.png align="center")

Enter your admin email, and the password twice - press that `Continue` button and you will be logged in as the administrator.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699532693542/55bb5d41-dade-47fd-a5bb-42783f8baa8a.png align="center")

## SAML - Security Assertion Markup Language

We will use SAML to manage the single sign-on (SSO) - it is one of the many standards you will come across among Leightweight Directory Access Protocol (LDAP), Open Authorization (OAuth2).

Our SIEM wazuh can use SAML or LDAP for external auth and since LDAP usually requires a service account with a password (pretty insecure if you ask me) we will use SAML.

## SAML Setup in authentik - user / group

First, I would suggest to create a dedicated user account, e.g. called `wazuh-admin` -

to do that click on `Directory -> Users` in the navigation on the left side of the dashboard.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533521531/41a34e47-60d7-454c-8a92-07ae0cd4a96c.png align="center")

Next up click the blue `Create` button in the middle of the screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533509127/7ddb86d7-a0db-48f9-b588-31f7a2b1eeeb.png align="center")

Add a `username` and leave the rest as default.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533817720/3e2f8ece-1cb4-4ec9-9728-53fffaeb8d7d.png align="center")

Then create a group and add the `wazuh-admin` user into it

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533969809/ee7545be-e909-4784-ba5b-b1d7fe370187.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699533944055/b6645b61-631d-4c44-82ba-5cf0237a3b1b.png align="center")

### authentik provider

Now is the time to create a provider - providers are essentially beacons for external applications to ask for the user details and validate the tokens the user provides after login.

We can find them under `Applications -> Providers` and once again clicking the big blue button in the middle of the screen brings us to the form to create a new one.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699534372247/1d38bba2-cfd8-4da5-b7c1-f22f87989217.png align="center")

We need to choose SAML as the provider type.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699534164773/278bf0a9-20ee-48ed-a7d0-80569bdf6854.png align="center")

Then give it a descriptive name - e.g. `SAML`

select

* Authentication Flow (e.g. `default-authentication-flow`)
    
* Authorization Flow (e.g. `default-provider-authorization-implicit-constent`)
    
* set the ACS URL (`https://<wazuh_ip_or_hostname>/_opendistro/_security/saml/acs`)
    
* issuer (`wazuh-saml`)
    
* Service Provider Binding - `Post`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699534619131/99687eab-d913-496e-a866-2fbc46285ad6.png align="center")

Leave all the rest in the `advanced protocol settings` as default for now and click on "Finish".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699536212063/ed3a9865-cd99-49a5-8c88-edbbe7f3f036.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699536220302/2acb16a2-d3f9-4794-9bea-c3b03f0457ee.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699536230073/27c5462c-115f-4c91-9a49-41c13541c4b9.png align="center")

Next step is a property mapping - this is a function that takes information from the authentik users (e.g. username, email, groups) and provides it to the external service (wazuh).

We will use this to map group memberships (e.g. `wazuh-admins`) as backend roles that are used for RBAC (Role-based Access Control) in wazuh.

Without further ado - here is how to create a property mapping - under `Customisation -> Property Mappings`. Select the type and add the following details:

Name: `wazuh property mapping`

SAML Attribute Role: `Roles`

Expression:

```python
if ak_is_group_member(request.user, name="wazuh-admins"):
  yield "wazuh-admin"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699536507966/8a2ad362-d9ed-4ef1-a6d4-78795604f294.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699536513360/238e1b8e-9fa6-4f52-aed8-27db99e13b95.png align="center")

## Certificate Setup

We want to secure our communication with SAML, and to do that we need a certificate that is ideally only used for the SAML setup.

Lucky for us authentik has an option to generate and import them directly - under `System -> Certificates` you can find the option to `Generate` a new one.

Give it a `name` and set the validity period as `365` days and click `Generate`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699538105448/10576e80-c78a-4feb-8617-fdc7215c74ec.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699538163363/8b324f27-5e2c-4c1e-bdb9-c2817ada722c.png align="center")

## Adjust SAML Provider

Select the SAML provider and then click the `Edit` button - Then under `Advanced protocol settings` select the correct `Signing Certificate` and make sure to also select the `wazuh property mapping` in the `Property mappings.`

Once that is done push the `Update` button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699538203465/22b24e7d-a7df-47b4-8cca-1ec74e0df04d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699538255784/f3be82ef-a88a-43d0-88ba-9453b13704cd.png align="center")

The last step on the authentik side is to create an application that uses our SAML provider.

## authentik application

You can do that via the nagivation bar `Applications -> Applications` - `Create` and setting the following parameters:

* Name: `wazuh-saml`
    
* Slug: `wazuh-saml`
    
* Provider: `SAML`
    
* Policy Engine: `any`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699538900121/361c7ca4-f199-4d49-b276-a7cfc9f8b00e.png align="center")

Leave the UI as default or upload a logo you would like to use to identify the application in the dashboard - e.g. [https://avatars.githubusercontent.com/u/13752566?s=200&v=4](https://avatars.githubusercontent.com/u/13752566?s=200&v=4)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699539097716/4d59f966-a2d5-4a69-bb29-9e7c7908b53e.png align="center")

The last step is to download the metadata file from the provider - or as an alternative copy the download url.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699553343216/963de318-d110-4a12-a9c0-5fa447ab4d2a.png align="center")

Nice - that wraps up the authentik part. Now to wazuh.

## wazuh setup for SAML

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">IF you have not yet installed wazuh in a Virtual machine - <a target="_blank" rel="noopener noreferrer nofollow" href="https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide" style="pointer-events: none">https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide</a></div>
</div>

The first file that we have to adjust is `/etc/wazuh-indexer/opensearch-security/config.yml` - open it with your favorite text editor (e.g. `nano`) and add the information below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699556780873/7eca678f-a701-4495-80be-1cbb117a85c5.png align="center")

```yaml
authc:
  basic_internal_auth_domain:
    description: "Authenticate SAML against internal users database"
    http_enabled: true
    transport_enabled: true 
    order: 0
    http_authenticator:
      type: basic 
      challenge: false
    authentication_backend:
      type: intern 
  saml_auth_domain:
    http_enabled: true
    transport_enabled: false
    order: 1
    http_authenticator:
      type: saml 
      challenge: true
      config: 
        idp:
          metadata_file: "/etc/wazuh-indexer/opensearch-security/idp-metadata.xml"
          entity_id: "wazuh-saml"
        sp:
          entity_id: "wazuh-saml"
        kibana_url: "https://<YOUR_WAZUH_IP_HOSTNAME>/"
        roles_key: Roles
        exchange_key: "MIIGBDCCA+SQs..."
    authentication_backend:
      type: noop
```

The lines that are variable are -

* `idp.metadata_file` - the location/name you gave the downloaded `metadata.xml` file - I would recommend putting it in `/etc/wazuh-indexer/opensearch-security/` and naming it `idp-metadata.xml`.
    
    * you now need to change ownership and rights on the file to make sure it is properly usable by wazuh / secure
        
    * ```bash
                    chown wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/opensearch-security/idp-metadata.xml
                    
                    chmod 640 /etc/wazuh-indexer/opensearch-security/idp-metadata.xml
        ```
        
* `idp.entity_id` - if you followed this guide it is `wazuh-saml` - you can also see it in the metadata file
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699557835881/43c50644-447c-4e13-b253-ce0ba0ee6c56.png align="center")
    
* `sp.entity_id` - if you followed this guide this is `wazuh-saml`
    
* `sp.kibana_url` - your wazuh dashboard url - e.g. `https://wazuh.mydomain.com` or `https://<IP_OF_YOUR_WAZUH_VM`
    
* `roles_key` - this is the name you entered into the role mapping - if you followed this guide it is called `Roles`
    
* and `exchange_key` - copy it from the metadata file - you can find it between the `<ds:X509Certificate>` and `</ds:X509Certificate>` tags, usually starts with `MII` - dont forget the `"` around the key
    

save the file and run the `securityadmin.sh` script from the following location:

```bash
export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /etc/wazuh-indexer/opensearch-security/config.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h localhost -nhnv
```

if all is well, it should finish with `Done with success` like below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699558756208/895334c7-b4f3-4ec9-a13c-a74437775cff.png align="center")

The next step is to adjust the `/etc/wazuh-indexer/opensearch-security/roles_mapping.yml`

open the file and scroll down until you see the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699558882658/2094c09c-98b5-4005-b620-7cb9f6e6c04f.png align="center")

Now, remember the roles mapping you created earlier? In it you defined a group and the corresponding backend role that should be returned - if you followed this tutorial - it is `wazuh-admin`.

This role needs to be added to the `roles_mapping.yml` now like below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699558859745/296a33d9-57a7-4a8b-ac86-fa2e65f7e178.png align="center")

Save the file, and run the `securityadmin.sh` again but this time with the `roles_mapping.yml` as the changed file.

```bash
export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /etc/wazuh-indexer/opensearch-security/roles_mapping.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h localhost -nhnv
```

once again, if all goes well it should return - `Done with success`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699561382371/429073c2-8e7e-49bd-8eea-b951f20925bf.png align="center")

Now the last three steps - first we check `/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml`

The line that interests us is the last one - `run_as` if that one is set to `true` we can change it to `false`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699561771704/51a1ff51-9510-4273-86d4-8867eb0e0252.png align="center")

The penultimate task is to add a role to wazuh - open the dashboard - click on the arrow next to the wazuh logo then on `Security` and `Roles mapping`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699562352409/787bda13-1390-455e-a530-f4304edf6f2a.png align="center")

We will now add a new role mapping - give it any descriptive name add the respective `Roles` -&gt; in this case `administrator` and add a new custom rule at the bottom that matches (`FIND`) the `user_name` to `wazuh-admin`.

Save the role mapping.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699562344492/ce69dd50-696f-41e9-867f-6c068299324b.png align="center")

The last step is to add saml authentication to `/etc/wazuh-dashboard/opensearch_dashboards.yml`

Add the following lines to the file:

```bash
opensearch_security.auth.type: "saml"
server.xsrf.allowlist: ["/_opendistro/_security/saml/acs", "/_opendistro/_security/saml/logout", "/_opendistro/_security/saml/acs/idpinitiated"]
opensearch_security.session.keepalive: false
```

after the change, the file could look something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699562718133/b4548d02-b338-4d74-b438-df965fe44a3d.png align="center")

now restart the wazuh-dashboard service and when you visit the wazuh dashboard you will be greeted by the authentik login.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699563048909/1c3f8b85-452b-4a50-8a7a-ae4dd5c223f6.png align="center")

if you login as the `wazuh-admin` user you will be forwarded to wazuh as `wazuh-admin` 🎉🎉🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699563156563/fff4942f-60d8-43e6-b12e-1d12af730650.png align="center")

*Thank you to Videothek for the brain teaser - this one took the better part of 4 weeks to figure out correctly because there was no documentation how to achieve this.*

If you like this content - you can check out [https://maikroservice.com/email](https://maikroservice.com/email) for more content like this.