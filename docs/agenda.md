# Agenda

1. Welcome and introduction to the workshop [Peter Gietz] (10 m)

2. Introduction to Shibboleth (Project, SP, IdP) [David Huebner] (20 m)
- Shibboleth project, short IdP introduction, SP introdcution

3. Federation tools (aggregator, discovery service [Rainer Hörbe] (20 m)
- Using pyFF as metadata aggregator. Configuration, use cases, security
- Discovery services: central/embedded, Shibboleth, pyFF
- Trust relationships in Federations
- Lab demo

4. Shib-IdP hosting as a DFN service [Wolfgang Pempe] (20 m)
- Without "as a DFN service"
- Examples, onboarding, common problems, experiences, ..

6. Plugin-Interfaces of Shib idP v3 [David] (30 m)
- Customizations that can be done with reasonable effort
- Examples: MFA, Interceptor, AD->Shib mapping of users, ..

7. Hands-on [David, Rainer, Wolfgang] (120 m)
- Install and configure Shib IdP 
- Register with federation
- Test a federation with two provided SPs
- Explore typical configuration errors and how to trace them (e.g. non registered entity ID, bad certificate, wrong attribute mapping, wrong attribute release)
