# Lab Exercises: Metadata

## Overview

Goals of this hands-on session:
- Get to know Shibboleth SP (and to some degree IdP) with regard to metadata and attribute management
- Understand metadata aggregation with practical examples
- Identify common errors and learn how to fix them

A VM with a pre-installed and pre-configured Shibboleth SP and IdP will be provided.
In addition there will be one metadata aggregation service that will collect, verify and aggregate the metadata from all participants.


## Lab components

![Compontents overview](lab1-components.png)


## Steps

### Step 1

#### Introduction to the infrastructure: 

* https://github.com/tiime2018/lab1-mdreg/blob/master/docs/MD_Registry.md
* See slides

#### Install and start test VM

* See slides for details

#### Install Shibboleth SP

* ```yum install shibboleth```
* ```systemctl enable shibd --now```
* ```systemctl restart httpd```

### Step 2

In order to allow the aggregation of the SPs and IdPs of all participants, every entity needs to have a unique entityID. The provided IdP on the VM uses a pre-configured "dummy entityID", that is the same on all instances, which will not work.

Therefore this step consists of two tasks:

#### Change entityID of the Shibboleth SP on your local VM

* Configuration in /etc/shibboleth/shibboleth2.xml
* Change entityID to something unique, e.g. https://localhost/shibboleth/XX
* XX has to be different for every participant

#### Change entityID of the Shibboleth IdP on your local VM

* Apply the same changed to the IdP configuration in /opt/shibboleth-idp/conf/idp.properties
* Use a similar entityID, e.g. https://localhost/idp/shibboleth/XX
* Also update the IdP metadata file to reflect the changed entityID in /opt/shibboleth-idp/metadata/idp-metadata.xml

### Step 3

To allow the central metadata aggregation service to combine all metadata files we will use a github repository to collect all metadata files.

Please follow the guidelines in https://github.com/tiime2018/lab1-mdreg/blob/master/docs/MD_Registry.md to verify and upload your metadata files.

#### Generate and save SP and IdP metadata 

#### Rename files accordingly

* e.g. userXX_sp.xml and userXX_idp.xml

#### Check validity

* Use the metadata validator at https://mdval.test.portalverbund.at and check both files with the saml2int0.2 profile
* There should be no errors
* Ignore the Warnings for now

#### Upload metadata files to federation (via github)

* Upload both files here: https://github.com/tiime2018/lab1-mdreg/tree/master/metadata/upload

### Step 4

This is done to showcase the common problem of outdated / non-matching credentials. You will generate new certificates for your local SP.

#### Generate new certificates

* use the keygen tool from Shibboleth SP: ```./etc/shibboleth/keygen.sh -f```

#### Update your SP to use these new certificates

* Just restart shibd
* DO NOT upload new metadata files yet!

### Step 5

Live Demo: Metadata Aggregation of metadata files of all participants with pyFF

### Step 6

After the aggregated metadata is ready, all participants should update their IdP and SP to load the federation metadata.

#### Configure Shibboleth SP to use provided Discovery Service and load aggregated metadata

* Download federation signing certificate from http://mdfeed.lab.tiimeworkshop.eu/metadata_crt.pem
* Change /etc/shibboleth/shibboleth2.xml accordingly:

```
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://ds.lab.tiimeworkshop.eu/role/idp.ds">
  SAML2
</SSO>
```

```
<MetadataProvider type="XML" validate="true"
  uri="http://mdfeed.lab.tiimeworkshop.eu/metadata.xml"
      backingFilePath="federation-metadata.xml" reloadInterval="7200">
    <MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
    <MetadataFilter type="Signature" certificate="metadata_crt.pem"/>
    <DiscoveryFilter type="Blacklist" matcher="EntityAttributes" trimTags="true" 
      attributeName="http://macedir.org/entity-category"
      attributeNameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"
      attributeValue="http://refeds.org/category/hide-from-discovery" />
</MetadataProvider>
```

#### Configure Shibboleth IdP to load aggregated metadata

* This is done in /opt/shibboleth-idp/conf/metadata-providers.xml

```
<MetadataProvider id="TIIME-Lab-Federation"
                  xsi:type="FileBackedHTTPMetadataProvider"
                  backingFile="%{idp.home}/metadata/localCopyFromTIIME-Lab-Federation.xml"
                  metadataURL="http://mdfeed.lab.tiimeworkshop.eu/metadata.xml">

    <MetadataFilter xsi:type="SignatureValidation" certificateFile="/etc/shibboleth/metadata_crt.pem" />
    <MetadataFilter xsi:type="RequiredValidUntil" maxValidityInterval="P30D"/>
    <MetadataFilter xsi:type="EntityRoleWhiteList">
        <RetainedRole>md:SPSSODescriptor</RetainedRole>
    </MetadataFilter>
</MetadataProvider>
```


### Step 7

In this step the SP and IdP will use the aggregated metadata to communicate with each other. 

Experiment: use the SP on your VM to try to login via your local IdP

Expected result: SP will *not* accept the assertion from the IdP. This is because the changed metadata (Step 4) is not yet reflected in federation metadata. 

Sidenote: You will also see, that the Discovery service does not display a proper name for your SP or IdP. To change this, it makes sense to also include UI-Elements in your metadata.

### Step 8

In order to fix the issues the federation metadata needs to be updated with the new certificates and extended with some human-readable info about your entities.

#### Edit metedata files

* Change the metadata generation handler in /etc/shibboleth/shibboleth2.xml to include contact information (SHOULD in saml2int) and the CoCo Entity Category:

```
<Handler type="MetadataGenerator" Location="/Metadata" signing="false">
    <md:ContactPerson contactType="support">
        <md:EmailAddress>userXX@localhost.de</md:EmailAddress>
    </md:ContactPerson>
    <md:ContactPerson contactType="technical">
        <md:EmailAddress>userXX@localhost.de</md:EmailAddress>
    </md:ContactPerson>
    <mdattr:EntityAttributes xmlns:mdattr="urn:oasis:names:tc:SAML:metadata:attribute">
        <saml:Attribute Name="http://macedir.org/entity-category" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
            <saml:AttributeValue>http://www.geant.net/uri/dataprotection-code-of-conduct/v1</saml:AttributeValue>
        </saml:Attribute>
    </mdattr:EntityAttributes>
</Handler>
```

* Change the IdP metadata in /opt/shibboleth-idp/metadata/idp-metadata.xml to include some UI info and the contact details:

```
<Extensions>
    <shibmd:Scope regexp="false">localhost</shibmd:Scope>
    <mdui:UIInfo>
        <mdui:DisplayName xml:lang="en">Lab IdP from user XX</mdui:DisplayName>
        <mdui:Description xml:lang="en">Some fancy description can go here</mdui:Description>
    </mdui:UIInfo>
</Extensions>
...
<ContactPerson contactType="support">
    <EmailAddress>userXX@localhost.de</EmailAddress>
</ContactPerson>
<ContactPerson contactType="technical">
    <EmailAddress>userXX@localhost.de</EmailAddress>
</ContactPerson>
```
#### Verify again

* Once again use the metadata valiadator at https://mdval.test.portalverbund.at/
* Everything should be OK now

### Upload via github

* Reupload (same filenames as before) your two metadata files: https://github.com/tiime2018/lab1-mdreg/tree/master/metadata/upload

### Step 9

Live demo: Repeat metadata aggregation.

### Step 10

#### Reload federation metadata in your local SP and IdP

* Quick and dirty: restart tomcat and shibd
* Or find a better way ... :)

#### Repeat the experiment in step 7.

This time everything should work out and the Discovery Service should display nice names.

However, the SP will only receive a very limited set of attributes.

### Step 11

The IdP needs to be reconfigured to release more attributes to your SP.
Additionally, the SP also needs to accept these attributes.

#### Configure attribute-filtering and -mapping in your SP and IdP

* First allow your IdP to release attributes to all CoCO SPs in /opt/shibboleth-idp/conf/attribute-filter.xml

```
<PolicyRequirementRule xsi:type="EntityAttributeExactMatch" 
                               attributeName="http://macedir.org/entity-category"
                               attributeValue="http://www.geant.net/uri/dataprotection-code-of-conduct/v1" /
```

* Accept some common attributes in /etc/shibboleth/attribute-map.xml
* Map uid to REMOTE_USER in /etc/shibboleth/shibboleth2.xml

### Step 12

Repeat Step 7


### Optional bonus steps

#### Consume expired metdata

##### Rough steps

* We will generate aggregated metadata that is already expired
* Reload metadata in your SP or/and IdP

##### Resultes to be observed

* UI error message
* Check log files for errors
* Discussion

#### Consume metdata with invalid signature certificate

##### Rough steps

* We will generate aggregated metadata with an invalid signature

##### Resultes to be observed

* UI error message
* Check log files for errors
* Discussion


