# TIIME 2018 IDM OSS Track 1 (Sibboleth for FO): Metadata Registry

For the purpose of the lab the Metadata administration is kept simple, 
using Github for upload and pyFF for aggregation.

## Process to register metadata in the test federation

* Create an EntityDescriptor and validate it using the saml2int profile: https://mdval.test.portalverbund.at
* Commit the EntityDescriptor to /metadata/upload/userXX_yourentity.xml in this repository;
* A Jenkins job will start pyff to produce the metadata aggregate;
* Metadata URLs:

  - [HTML overview](http://mdfeed.lab.tiimeworkshop.eu])
  - [Aggregate: http://mdfeed.lab.tiimeworkshop.eu/metadata.xml](http://mdfeed.lab.tiimeworkshop.eu/metadata.xml)
  - [Manage aggregator (requires authorization)](https://jenkins.lab.tiimeworkshop.eu/metadata.xml)
  
  
## Components and their responsibilities 

The lab setup is simplified. In particular it is using a software certificate to sign the metadata,
which should be done with a HW-token in a production system. 
Also having a maching for signing separate from the internet-facing service would be recommended.
 
* pyFF "Federation Feed". This is a metadata aggregator, plus a MDQ and an IDP-discovery service.
  It is packaged as docker container. We are using a yet-to-be-merged fork with a few additions.
  The deplyoment uses the github repo identinetics/d-pyff.

* HTTPS front-end. Assuming that in a typical deployment all TLS-certificates would be centralized at a certain point such as a load balancer,
  this setup uses an NGINX instance to proxy HTTPS and serve static contents.
  The deplyoment is using the github repo identinetics/d-nginx.

* Metadata Validator. Provides syntactical validation (XML Schema + various schematron rules) 
  
* Jenkins is triggered to execute pyff to pull the metadata from this repo and start the metadata aggregator.
  Users can view pyff logs files in the Jenkins web interface which helps to understand problems.

* There are an IDP and SP to validate the basic setup of the federation:

  - [Test SP: echo.lab.tiimeworkshop.eu](https://echo.lab.tiimeworkshop.eu)