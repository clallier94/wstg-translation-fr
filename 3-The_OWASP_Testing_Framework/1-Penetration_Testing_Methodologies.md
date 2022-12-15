# Méthodologies de test d'intrusion

## Sommaire

- [Guides de test OWASP](#guides-de-test-owasp)
    - [OWASP Guide de test de sécurité Web](https://owasp.org/www-project-web-security-testing-guide/)
    - [OWASP Guide de test de sécurité mobile](https://owasp.org/www-project-mobile-security-testing-guide/)
    - [OWASP Méthodologie de test de sécurité du micrologiciel](https://github.com/scriptingxss/owasp-fstm)
- [Norme d'exécution des tests d'intrusion](#norme-dex%C3%A9cution-des-tests-dintrusion)
- [Guide des tests d'intrusion PCI](#guide-de-test-de-p%C3%A9n%C3%A9tration-pci)
    - [Guide des tests de pénétration PCI DSS](#guide-des-tests-de-p%C3%A9n%C3%A9tration-pci-dss)
    - [Exigences des tests d'intrusion PCI DSS](#exigences-des-tests-dintrusion-pci-dss)
- [Cadre de test d'intrusion](#cadre-de-test-dintrusion)
- [Guide technique des tests et de l'évaluation de la sécurité de l'information](#guide-technique-des-tests-et-de-l%C3%A9valuation-de-la-s%C3%A9curit%C3%A9-de-linformation)
- [Manuel de méthodologie de test de sécurité Open Source](#manuel-de-m%C3%A9thodologie-de-test-de-s%C3%A9curit%C3%A9-open-source)
- [Références](#r%C3%A9f%C3%A9rences)

## Guides de test OWASP

En termes d'exécution des tests de sécurité technique, les guides de test OWASP sont fortement recommandés. Selon les types d'applications, les guides de test sont répertoriés ci-dessous pour les services Web/cloud, l'application mobile (Android/iOS) ou le micrologiciel IoT, respectivement.

- [OWASP Guide de test de sécurité Web](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Guide de test de sécurité mobile](https://owasp.org/www-project-mobile-security-testing-guide/)
- [OWASP Méthodologie de test de sécurité du micrologiciel](https://github.com/scriptingxss/owasp-fstm)

## Norme d'exécution des tests d'intrusion

La norme d'exécution des tests d'intrusion (PTES) définit les tests d'intrusion en 7 phases. En particulier, les directives techniques PTES donnent des suggestions pratiques sur les procédures de test et des recommandations pour les outils de test de sécurité.

- Interactions pré-engagement
- La collecte de renseignements
- Modélisation des menaces
- Analyse de vulnérabilité
- Exploitation
- Post Exploitation
- Rapports

[Directives techniques PTES](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

## Guide de test de pénétration PCI

L'exigence 11.3 de la norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS) définit les tests d'intrusion. PCI définit également les directives sur les tests de pénétration.

### Guide des tests de pénétration PCI DSS

La directive sur les tests de pénétration PCI DSS fournit des conseils sur les points suivants :

- Composants de test de pénétration
- Qualifications d'un testeur d'intrusion
- Méthodologies de test d'intrusion
- Lignes directrices pour les rapports sur les tests d'intrusion

### Exigences des tests d'intrusion PCI DSS

L'exigence PCI DSS fait référence à l'exigence 11.3 de la norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS)

- Basé sur des approches acceptées par l'industrie
- Couverture pour CDE et systèmes critiques
- Comprend des tests externes et internes
- Test pour valider la réduction de périmètre
- Tests de la couche applicative
- Tests de la couche réseau pour le réseau et le système d'exploitation

[Guide des tests de pénétration PCI DSS](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)

## Cadre de test d'intrusion

Le cadre de test d'intrusion (PTF) fournit un guide pratique complet sur les tests d'intrusion. Il répertorie également les utilisations des outils de test de sécurité dans chaque catégorie de test. Le domaine principal des tests d'intrusion comprend :

- Empreinte Réseau (Reconnaissance)
- Découverte & Sondage
- Énumération
- Craquage de mot de passe
- Évaluation de la vulnérabilité
- Audit AS/400
- Tests spécifiques Bluetooth
- Tests spécifiques Cisco
- Tests spécifiques à Citrix
- Dorsale du réseau
- Tests spécifiques au serveur
- Sécurité VoIP
- Pénétration sans fil
- Sécurité physique
- Rapport final - modèle

[Cadre de test d'intrusion](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)

## Guide technique des tests et de l'évaluation de la sécurité de l'information

Le guide technique des tests et de l'évaluation de la sécurité de l'information (NIST 800-115) a été publié par le NIST, il comprend certaines techniques d'évaluation énumérées ci-dessous.

- Techniques de révision
- Techniques d'identification et d'analyse de cibles
- Techniques de validation des vulnérabilités cibles
- Planification de l'évaluation de la sécurité
- Exécution de l'évaluation de la sécurité
- Activités post-test

Le NIST 800-115 est accessible [ici](https://csrc.nist.gov/publications/detail/sp/800-115/final)

## Manuel de méthodologie de test de sécurité Open Source

Le manuel de méthodologie de test de sécurité Open Source (OSSTMM) est une méthodologie pour tester la sécurité opérationnelle des emplacements physiques, le flux de travail, les tests de sécurité humaine, les tests de sécurité physique, les tests de sécurité sans fil, les tests de sécurité des télécommunications, les tests de sécurité des réseaux de données et la conformité. L'OSSTMM peut prendre en charge la référence ISO 27001 au lieu d'un guide de test de pénétration d'application pratique ou technique.

L'OSSTMM comprend les sections clés suivantes :

- Analyse de sécurité
- Métriques de sécurité opérationnelle
- Analyse de confiance
- Flux de travail
- Tests de sécurité humaine
- Tests de sécurité physique
- Test de sécurité sans fil
- Tests de sécurité des télécommunications
- Tests de sécurité des réseaux de données
- Règlement de conformité
- Reporting avec le STAR (Security Test Audit Report)

[Manuel de méthodologie de test de sécurité Open Source](https://www.isecom.org/OSSTMM.3.pdf)

## Références

- [Norme de sécurité des données PCI – Guide des tests d'intrusion](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
- [Norme PTES](http://www.pentest-standard.org/index.php/Main_Page)
- [Manuel de méthodologie de test de sécurité Open Source (OSSTMM)](http://www.isecom.org/research/osstmm.html)
- [Guide technique des tests et de l'évaluation de la sécurité de l'information NIST SP 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [Évaluation des tests de sécurité HIPAA 2012](http://csrc.nist.gov/news_events/hiipaa_june2012/day2/day2-6_kscarfone-rmetzer_security-testing-assessment.pdf)
- [Cadre de test de pénétration 0.59](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- [Guide de test de sécurité mobile OWASP](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Directives de test de sécurité pour les applications mobiles](https://owasp.org/www-pdf-archive/Security_Testing_Guidelines_for_mobile_Apps_-_Florian_Stahl+Johannes_Stroeher.pdf)
- [Kali Linux](https://www.kali.org/)
- [Supplément d'information : Exigence 11.3 Test de pénétration](https://www.pcisecuritystandards.org/pdfs/infosupp_11_3_penetration_testing.pdf)
