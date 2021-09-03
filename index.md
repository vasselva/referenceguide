## Security Reference materials

- [OWASP top 10 2017](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)
- [MITRE attack framework](https://attack.mitre.org/)
- [Cloud Security Vulnerabilities](https://media.defense.gov/2020/Jan/22/2002237484/-1/-1/0/CSI-MITIGATING-CLOUD-VULNERABILITIES_20200121.PDF)
- [NIST container security guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf)
- [PKI explained clearly](https://smallstep.com/blog/everything-pki/)

## Identity and Access Management
- [ OAuth explained-1](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc)
- [ OAuth explained-2](https://www.youtube.com/watch?v=PfvSD6MmEmQ&list=PLNYaEAlwFjtwo6IqfBRr7-z2PuCE4ER8Q)

## Vault

- [vault jwt backend](./vault/vault_jwt.md) // Internal
- [Vault cert-manager Integration](./vault/vault_certmanager.md) // Internal
- [Vault AppRole Authentication](https://learn.hashicorp.com/vault/identity-access-management/iam-authentication)

## Containers

- [ Google Container security](https://cloud.google.com/containers/security/)
- [ Pod Security Policies](./k8s.md)

## AWS

- [AWS re-invent videos](https://reinventvideos.com/) 
- [AWS Local Lambda](https://github.com/lambci/docker-lambda/)
- [Lambda Dockerfile](./docker.md)
- [Secure API gateway Lambda Authoriser - CDK ](https://github.com/vasselva/secure-api-gateway-auth0-lambda-custom-authorizer.git) // Internal

## Secure Configuration

- [Automation of Hardening baselines](https://github.com/dev-sec) 

## Certification

- [ CISSP Pratice test - Boson](https://www.boson.com/practice-exam/cissp-isc2-practice-exam)

## Useful Commands

*pandoc csv to md conversion*

```pandoc.exe test.csv -o test.md -f csv -t gfm-hard_line_breaks-pipe_tables+ascii_identifiers-autolink_bare_uris+gfm_auto_identifiers+raw_html  --columns=999 --wrap=none```
