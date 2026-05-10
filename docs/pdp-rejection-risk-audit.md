# Faktur - audit des risques de rejet plateforme agreee

Date: 2026-04-24
Objet: identifier ce qui peut faire rejeter ou fragiliser une demande d'immatriculation comme plateforme agreee.

## Conclusion courte

Ne pas envoyer de dossier d'immatriculation maintenant. Le code contient des briques utiles, mais Faktur n'a pas encore les preuves techniques, securite, interop et exploitation attendues pour etre immatricule comme plateforme agreee. Le risque principal n'est pas un champ manquant sur une facture: c'est l'ecart entre ce qui serait annonce dans le dossier et ce que l'application prouve reellement aujourd'hui.

Sources officielles de cadrage:

- https://www.impots.gouv.fr/facturation-electronique-et-plateformes-agreees
- https://www.impots.gouv.fr/liste-des-pieces-fournir-pour-demander-devenir-une-plateforme-agreee
- https://www.impots.gouv.fr/liste-des-plateformes-agreees-immatriculees
- https://www.impots.gouv.fr/specifications-externes-b2b
- https://www.impots.gouv.fr/guide-utilisateur-de-la-demarche-dimmatriculation-du-site-demarches-simplifiees
- https://entreprendre.service-public.gouv.fr/actualites/A16585

## Bloquants probables

### 1. Faktur ne prouve pas qu'elle est une plateforme agreee autonome

Preuve code:

- `apps/backend/app/services/einvoicing/pdp_service.ts:3` limite les providers a `b2brouter` et `sandbox`.
- `apps/backend/app/services/einvoicing/pdp_service.ts:39` force `sandbox` sans cle API, sinon `b2brouter`.
- `apps/backend/app/services/einvoicing/pdp_service.ts:143` et `:160` appellent seulement l'API B2Brouter.
- `apps/backend/start/routes/einvoicing.ts:10` et `:11` exposent seulement soumission et validation de connexion.

Risque dossier: si Faktur demande l'immatriculation en son nom, le fait de s'appuyer uniquement sur B2Brouter ne demontre pas la capacite propre de Faktur a emettre, recevoir, router, transmettre les donnees et gerer les statuts selon les specifications officielles. B2Brouter peut etre un prestataire ou connecteur, mais il faut le documenter comme tel et prouver le perimetre exact de responsabilite.

Action avant dossier:

- definir si l'objectif est `Faktur plateforme agreee` ou `Faktur logiciel compatible connecte a une plateforme agreee`;
- si plateforme agreee: developper les connecteurs officiels, flux entrants/sortants, statuts, rejets, correction, reprise sur incident et preuves d'interoperabilite;
- si logiciel compatible: ne pas utiliser la promesse "Faktur est plateforme agreee".

### 2. E-reporting absent

Preuve code:

- aucun module dedie `e-reporting`, `ereporting`, `transaction reporting`, `annuaire` ou `concentrateur` n'apparait dans `apps/backend/app`.
- `apps/backend/database/migrations/0010_create_invoice_settings_table.ts:29-32` ne stocke que activation e-invoicing et configuration PDP, pas les obligations d'e-reporting.

Risque dossier: l'administration attend la transmission des donnees de transactions et de paiement dans les cas hors facturation electronique. Aujourd'hui Faktur n'a pas de collecte B2C/international, pas d'agregation, pas de calendrier de declaration, pas de corrections, pas de preuve d'envoi.

Action avant dossier:

- modeliser les periodes de reporting, regimes TVA, categories d'operation et moyens de paiement;
- generer les flux XML/JSON conformes aux specifications externes;
- ajouter planification, relance, idempotence, suivi des rejets et corrections.

### 3. Annuaire central absent

Preuve code:

- aucune table ou route ne gere les points de reception, inscriptions annuaire, mises a jour ou desactivations.

Risque dossier: une plateforme agreee doit alimenter les informations utiles a l'annuaire et gerer le routage des destinataires. Sans ce module, le dossier ne peut pas prouver que Faktur sait maintenir les participants et points de reception.

Action avant dossier:

- creer le modele `platform_participants` avec SIREN/SIRET, role, point de reception, plateforme choisie, dates d'activation/desactivation;
- creer les exports/API annuaire et les preuves de synchronisation.

### 4. Factur-X annonce mais PDF/A-3 non fabrique

Preuve code:

- `apps/backend/app/services/pdf/pdf_generator.ts:3-21` genere un PDF via Puppeteer.
- `apps/backend/app/services/pdf/document_pdf_service.ts:231` retourne ce PDF simple.
- `apps/backend/app/controllers/quote/export/pdf.ts:99-101` calcule un XML Factur-X mais le place seulement dans des headers `X-Facturx-*`; il n'est pas incorpore dans le PDF.
- `apps/backend/start/routes/quote.ts:27` expose un XML Factur-X separe pour les devis; il n'existe pas de route equivalente pour les factures.

Risque dossier: promettre un `PDF/A-3 Factur-X` serait incoherent. Un vrai Factur-X est un PDF/A-3 avec XML embarque et metadata conformes. Ici Faktur produit un PDF visuel et un XML CII de travail, pas un conteneur Factur-X probant.

Action avant dossier:

- ajouter une chaine PDF/A-3 avec attachement XML `factur-x.xml`, metadata XMP et validation;
- ou retirer toute promesse PDF/A-3 tant que ce n'est pas fait.

### 5. Validation officielle insuffisante

Preuve code:

- `apps/backend/app/services/einvoicing/pdp_service.ts:100-128` valide le XML par recherche de chaines (`includes`) seulement.

Risque dossier: les tests d'interoperabilite et les specs externes demandent une validation de formats et de regles metier. Une validation par presence de texte ne prouve pas la conformite EN16931, Factur-X/CII, UBL, ou aux regles francaises.

Action avant dossier:

- integrer XSD + Schematron officiels;
- conserver les rapports de validation;
- ajouter des jeux de tests de factures acceptees/rejetees.

### 6. UBL absent

Preuve code:

- aucune implementation `generateUblXml`, UBL 2.1, schema UBL ou export UBL n'a ete trouvee.

Risque dossier: si le dossier annonce le support UBL, il sera faux. Il faut soit limiter le perimetre annonce a CII/Factur-X, soit implementer UBL completement.

Action avant dossier:

- choisir les formats officiellement supportes;
- si UBL est annonce, ajouter generateur, validation et tests d'interoperabilite.

### 7. Cachet electronique qualifie / signature probante absent

Preuve code:

- `signatureField` est un booleen d'affichage dans les modeles facture/devis, pas une signature electronique.
- aucun service de certificat qualifie, HSM/KMS, horodatage, PAdES/XAdES ou cachet qualifie n'a ete trouve.

Risque dossier: un champ visuel de signature peut etre confondu avec une preuve d'authenticite. Il ne faut jamais le presenter comme un cachet electronique qualifie.

Action avant dossier:

- confirmer le mecanisme attendu selon le format retenu;
- integrer un prestataire de confiance qualifie si necessaire;
- documenter cycle de vie certificat, horodatage, revocation et preuves.

### 8. Archivage fiscal 6 ans non probant

Preuve code:

- pas de service de retention fiscale, stockage WORM/Object Lock, empreinte, scellement ou export controle fiscal.
- l'export d'equipe (`apps/backend/app/services/team/export_service.ts`) est un export applicatif, pas une archive probante.

Risque dossier: garder des donnees en base ne suffit pas pour prouver conservation, integrite, lisibilite et restitution sur la duree legale.

Action avant dossier:

- stocker chaque facture emise/recu, XML, statuts, preuves et journaux dans une archive horodatee;
- definir retention, purge legale, restitution et controle d'integrite periodique.

### 9. Journal d'audit non immuable

Preuve code:

- `apps/backend/database/migrations/0004_create_audit_logs_table.ts:7-18` cree une table classique.
- `apps/backend/app/services/audit/audit_log_service.ts:14-25` insere les evenements sans chaine de hash, signature, contrainte append-only ou export securise.

Risque dossier: un audit log modifiable en base est utile en developpement, mais insuffisant comme preuve forte pour une plateforme agreee.

Action avant dossier:

- ajouter hash chain, signature serveur, stockage append-only, retention et surveillance d'integrite;
- journaliser aussi les lectures/export de donnees sensibles, les erreurs interop, les rejets et corrections.

### 10. Securite/certifications non prouvees

Preuve repo:

- pas de certificat ISO/IEC 27001 dans le repo;
- pas d'engagement d'exploitation UE;
- pas de preuve hebergeur SecNumCloud;
- pas de dossier RGPD article 32 complet;
- pas de rapport d'audit independant ni tests d'interoperabilite.

Risque dossier: ces pieces sont administratives et organisationnelles. Le code ne peut pas les remplacer. Sans justificatifs, le dossier est incomplet meme si les champs facture sont corrects.

Action avant dossier:

- lancer ISO 27001 avec perimetre clair;
- choisir hebergement compatible avec les exigences;
- formaliser registre RGPD, analyse de risques, gestion incidents, sous-traitants, sauvegardes, PCA/PRA;
- planifier audit independant et tests officiels.

## Incoherences a corriger avant toute communication

- Eviter "Faktur est agreee" tant que la liste officielle ne mentionne pas Faktur.
- Eviter "envoi DGFiP/PPF direct" tant que le connecteur officiel n'existe pas.
- Eviter "PDF/A-3 Factur-X" tant que le XML n'est pas incorpore dans un PDF/A-3 valide.
- Eviter "suivi temps reel" tant que les statuts ne sont pas stockes, verifies periodiquement et exposes.
- Eviter "e-reporting" comme fonctionnalite livree tant que les flux de donnees et paiements n'existent pas.
- Eviter "cachet qualifie" tant qu'il n'y a pas de prestataire/certificat/protocole de signature qualifie.
- Expliquer le zero-access: la plateforme doit pouvoir extraire et transmettre des donnees legales. Si les donnees sont decryptables seulement pendant une session utilisateur, les jobs automatiques e-reporting, annuaire, paiement et relance statuts seront bloques.

## Ordre de travail recommande

1. Clarifier la strategie: plateforme agreee autonome ou logiciel compatible branche a une plateforme agreee.
2. Retirer toutes les promesses produit non prouvees.
3. Implementer un vrai paquet Factur-X/PDF-A-3 ou limiter le perimetre au XML CII de travail.
4. Ajouter validation officielle XSD/Schematron et jeux de tests.
5. Construire e-reporting, annuaire, statuts, corrections, reception, routage et jobs planifies.
6. Mettre en place archive probante et audit log immuable.
7. Lancer ISO 27001, hebergement UE/SecNumCloud, RGPD, audit independant.
8. Demander seulement un cadrage DGFiP tant que les preuves ne sont pas assemblees.

## Position recommandee pour un mail maintenant

Le mail ne doit pas dire "nous sommes prets" ni "nous demandons l'immatriculation". Il doit dire que Faktur prepare une candidature et demande confirmation du jeu de pieces, des specifications et de la procedure de tests applicable en 2026.
