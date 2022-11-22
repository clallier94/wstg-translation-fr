# M�thodologies de test d'intrusion

## Sommaire

- [Guides de test OWASP](#Guides-de-test-OWASP)
    - Guide de test de s�curit� Web (WSTG)
    - Guide de test de s�curit� mobile (MSTG)
    - M�thodologie de test de s�curit� du micrologiciel
- [Norme d'ex�cution des tests d'intrusion](#Norme-d'ex�cution-des-tests-d'intrusion)
- [Guide des tests d'intrusion PCI](#Guide-des-tests-d'intrusion-PCI)
    - [Guide des tests de p�n�tration PCI DSS](#Guide-des-tests-de-p�n�tration-PCI-DSS)
    - [Exigences des tests d'intrusion PCI DSS](#Exigences-des-tests-d'intrusion-PCI-DSS)
- [Cadre de test d'intrusion](#Cadre-de-test-d'intrusion)
- [Guide technique des tests et de l'�valuation de la s�curit� de l'information](#Guide-technique-des-tests-et-de-l'�valuation-de-la-s�curit�-de-l'information)
- [Manuel de m�thodologie de test de s�curit� Open Source](#Manuel-de-m�thodologie-de-test-de-s�curit�-Open-Source)
- [R�f�rences](#R�f�rences)

## Guides de test OWASP

En termes d'ex�cution des tests de s�curit� technique, les guides de test OWASP sont fortement recommand�s. Selon les types d'applications, les guides de test sont r�pertori�s ci-dessous pour les services Web/cloud, l'application mobile (Android/iOS) ou le micrologiciel IoT, respectivement.

- [OWASP Guide de test de s�curit� Web](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Guide de test de s�curit� mobile](https://owasp.org/www-project-mobile-security-testing-guide/)
- [OWASP M�thodologie de test de s�curit� du micrologiciel](https://github.com/scriptingxss/owasp-fstm)

## Norme d'ex�cution des tests d'intrusion

La norme d'ex�cution des tests d'intrusion (PTES) d�finit les tests d'intrusion en 7 phases. En particulier, les directives techniques PTES donnent des suggestions pratiques sur les proc�dures de test et des recommandations pour les outils de test de s�curit�.

- Interactions pr�-engagement
- La collecte de renseignements
- Mod�lisation des menaces
- Analyse de vuln�rabilit�
- Exploitation
- Post Exploitation
- Rapports

[Directives techniques PTES](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

## Guide de test de p�n�tration PCI

L'exigence 11.3 de la norme de s�curit� des donn�es de l'industrie des cartes de paiement (PCI DSS) d�finit les tests d'intrusion. PCI d�finit �galement les directives sur les tests de p�n�tration.

### Guide des tests de p�n�tration PCI DSS

La directive sur les tests de p�n�tration PCI DSS fournit des conseils sur les points suivants�:

- Composants de test de p�n�tration
- Qualifications d'un testeur d'intrusion
- M�thodologies de test d'intrusion
- Lignes directrices pour les rapports sur les tests d'intrusion

### Exigences des tests d'intrusion PCI DSS

L'exigence PCI DSS fait r�f�rence � l'exigence 11.3 de la norme de s�curit� des donn�es de l'industrie des cartes de paiement (PCI DSS)

- Bas� sur des approches accept�es par l'industrie
- Couverture pour CDE et syst�mes critiques
- Comprend des tests externes et internes
- Test pour valider la r�duction de p�rim�tre
- Tests de la couche applicative
- Tests de la couche r�seau pour le r�seau et le syst�me d'exploitation

[Guide des tests de p�n�tration PCI DSS](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)

## Cadre de test d'intrusion

Le cadre de test d'intrusion (PTF) fournit un guide pratique complet sur les tests d'intrusion. Il r�pertorie �galement les utilisations des outils de test de s�curit� dans chaque cat�gorie de test. Le domaine principal des tests d'intrusion comprend�:

- Empreinte R�seau (Reconnaissance)
- D�couverte & Sondage
- �num�ration
- Craquage de mot de passe
- �valuation de la vuln�rabilit�
- Audit AS/400
- Tests sp�cifiques Bluetooth
- Tests sp�cifiques Cisco
- Tests sp�cifiques � Citrix
- Dorsale du r�seau
- Tests sp�cifiques au serveur
- S�curit� VoIP
- P�n�tration sans fil
- S�curit� physique
- Rapport final - mod�le

[Cadre de test d'intrusion](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)

## Guide technique des tests et de l'�valuation de la s�curit� de l'information

Le guide technique des tests et de l'�valuation de la s�curit� de l'information (NIST 800-115) a �t� publi� par le NIST, il comprend certaines techniques d'�valuation �num�r�es ci-dessous.

- Techniques de r�vision
- Techniques d'identification et d'analyse de cibles
- Techniques de validation des vuln�rabilit�s cibles
- Planification de l'�valuation de la s�curit�
- Ex�cution de l'�valuation de la s�curit�
- Activit�s post-test

Le NIST 800-115 est accessible [ici](https://csrc.nist.gov/publications/detail/sp/800-115/final)

## Manuel de m�thodologie de test de s�curit� Open Source

Le manuel de m�thodologie de test de s�curit� Open Source (OSSTMM) est une m�thodologie pour tester la s�curit� op�rationnelle des emplacements physiques, le flux de travail, les tests de s�curit� humaine, les tests de s�curit� physique, les tests de s�curit� sans fil, les tests de s�curit� des t�l�communications, les tests de s�curit� des r�seaux de donn�es et la conformit�. L'OSSTMM peut prendre en charge la r�f�rence ISO 27001 au lieu d'un guide de test de p�n�tration d'application pratique ou technique.

L'OSSTMM comprend les sections cl�s suivantes�:

- Analyse de s�curit�
- M�triques de s�curit� op�rationnelle
- Analyse de confiance
- Flux de travail
- Tests de s�curit� humaine
- Tests de s�curit� physique
- Test de s�curit� sans fil
- Tests de s�curit� des t�l�communications
- Tests de s�curit� des r�seaux de donn�es
- R�glement de conformit�
- Reporting avec le STAR (Security Test Audit Report)

[Manuel de m�thodologie de test de s�curit� Open Source](https://www.isecom.org/OSSTMM.3.pdf)

## R�f�rences

- [Norme de s�curit� des donn�es PCI�� Guide des tests d'intrusion](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
- [Norme PTES](http://www.pentest-standard.org/index.php/Main_Page)
- [Manuel de m�thodologie de test de s�curit� Open Source (OSSTMM)](http://www.isecom.org/research/osstmm.html)
- [Guide technique des tests et de l'�valuation de la s�curit� de l'information NIST SP 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [�valuation des tests de s�curit� HIPAA 2012] (http://csrc.nist.gov/news_events/hiipaa_june2012/day2/day2-6_kscarfone-rmetzer_security-testing-assessment.pdf)
- [Cadre de test de p�n�tration 0.59](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- [Guide de test de s�curit� mobile OWASP](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Directives de test de s�curit� pour les applications mobiles](https://owasp.org/www-pdf-archive/Security_Testing_Guidelines_for_mobile_Apps_-_Florian_Stahl+Johannes_Stroeher.pdf)
- [Kali Linux] (https://www.kali.org/)
- [Suppl�ment d'information�: Exigence 11.3 Test de p�n�tration](https://www.pcisecuritystandards.org/pdfs/infosupp_11_3_penetration_testing.pdf)
