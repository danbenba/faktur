# Faktur - etat du dossier plateforme agreee

Date: 2026-04-24

## Point de realite

Faktur ne peut pas etre declare "plateforme agreee" par simple changement de code. La DGFiP demande une immatriculation sur dossier, des justificatifs de securite, des tests techniques/interoperabilite et un suivi d'audit. Une solution non immatriculee peut etre une solution compatible, mais elle ne peut pas transmettre elle-meme les factures, donnees de transaction et donnees de paiement a l'administration.

Sources officielles a joindre au dossier de travail:

- https://www.impots.gouv.fr/facturation-electronique-et-plateformes-agreees
- https://www.impots.gouv.fr/liste-des-pieces-fournir-pour-demander-devenir-une-plateforme-agreee
- https://www.impots.gouv.fr/liste-des-plateformes-agreees-immatriculees
- https://entreprendre.service-public.gouv.fr/actualites/A16585

## Ce qui vient d'etre couvert dans le code

- Champ facture `vatOnDebits` et parametre par defaut `defaultVatOnDebits`.
- Persistance API, validation et serialization du champ "TVA sur les debits".
- Saisie frontend de la nature d'operation et de la TVA sur les debits quand l'e-facturation est activee.
- Export Factur-X enrichi avec:
  - nature d'operation;
  - adresse de livraison dans `ShipToTradeParty`;
  - note "TVA acquittee d apres les debits".
- Soumission e-invoicing corrigee pour travailler sur les factures, pas sur les devis.
- Journalisation d'audit des actions facture critiques: creation, modification, changement de statut, paiement, export PDF, validation et soumission e-invoicing.
- Texte interface corrige pour ne pas promettre abusivement un envoi direct DGFiP ou une compatibilite avec toutes les plateformes agreees.

## Ce qui manque encore avant candidature

- SIREN de la societe candidate a fournir dans le dossier.
- Certification ISO/IEC 27001 dans le perimetre exact Faktur.
- Engagement d'exploitation du SI dans l'UE, sans transfert hors UE.
- Qualification SecNumCloud de l'hebergeur si externalise.
- Dossier RGPD article 32: mesures de securite, registre, sous-traitants, analyse de risques, gestion des incidents.
- Rapport d'audit de conformite par organisme independant.
- Tests techniques d'interoperabilite avec le PPF et avec une autre plateforme agreee.
- Annuaire central: alimentation, mise a jour, desactivation et points de reception.
- E-reporting complet: transactions B2C/internationales, agregats, paiements, corrections et echeances.
- Signature/cachet qualifie et/ou mecanisme officiel attendu selon les formats retenus.
- Archivage probant 6 ans avec politique de retention, preuve d'integrite et export controle fiscal.
- Validation XSD/Schematron officielle Factur-X/CII/UBL, pas seulement les controles de structure actuels.

## Mail court possible

Ce mail est uniquement une demande de cadrage. Il ne doit pas etre envoye comme une candidature ni laisser entendre que Faktur est deja pret pour l'immatriculation.

Objet: Preparation candidature plateforme agreee - Faktur

Bonjour,

Nous preparons une candidature pour l'immatriculation de Faktur comme plateforme agreee de facturation electronique. Nous souhaitons confirmer le dernier jeu de pieces et les modalites de tests d'interoperabilite applicables a une nouvelle candidature en 2026.

Faktur dispose deja d'un socle applicatif de preparation: donnees SIREN client, adresse de livraison, nature d'operation, mention TVA sur les debits, XML CII/Factur-X de travail, journalisation applicative et integration prestataire configurable. Nous sommes en train de cadrer les volets encore bloquants: ISO 27001, hebergement UE/SecNumCloud, RGPD, annuaire central, e-reporting, validation officielle, PDF/A-3 Factur-X et tests d'interoperabilite.

Pouvez-vous nous indiquer la procedure a suivre et les documents techniques a utiliser pour engager officiellement le dossier ?

Cordialement,
