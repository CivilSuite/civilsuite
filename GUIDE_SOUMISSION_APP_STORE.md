# CivilSuite — guide de soumission App Store, pas à pas

Version 1.0 (build 5) · iPhone et iPad · 20 localisations · guide rédigé le 24 octobre 2026

Ce guide suppose que vous partez de zéro sur chaque écran. Faites les étapes dans l'ordre : certaines dépendent des précédentes (le build doit être envoyé avant d'être associé à la version, le site doit être en ligne avant de saisir les URL).

Les libellés des menus Apple changent de temps en temps. Quand un libellé ne correspond pas exactement, cherchez le plus proche : la logique reste la même. Les libellés cités entre « » viennent de l'aide officielle d'App Store Connect consultée le 24/10/2026 (sources en fin de guide).

---

## 0. Ce que vous recevez

| Fichier / dossier | Contenu | Usage |
|---|---|---|
| `CivilSuite/` | Projet Xcode audité et mis à jour | Remplace votre dossier actuel |
| `site_a_publier/` | Site complet : 14 langues × (confidentialité, conditions, assistance) + aide PDF (fr, en) + page d'accueil + page 404 | À déposer tel quel sur GitHub Pages |
| `CivilSuite_AppStore/CivilSuite_AppStore_Textes.xlsx` | Tous les textes à copier-coller, avec compteurs de caractères | Saisie dans App Store Connect |
| `CivilSuite_AppStore/metadata/<langue>/` | Les mêmes textes en fichiers .txt, un dossier par langue | Copier-coller depuis un éditeur de texte si vous préférez |
| `CivilSuite_AppStore/_sources/` | Scripts qui régénèrent tout (`build.py`, `make_help_pdf.py`) | Seulement si un texte doit changer |
| Ce guide | La marche à suivre | — |

### Ce qui a changé dans l'app (résumé de l'audit)

Fonctions conservées à l'identique : iCloud Drive, WeatherKit, achat à vie 12,99 €, abonnement mensuel 0,99 € et annuel 9,99 € (essai 3 jours pour les deux abonnements). Les identifiants produits ne changent pas : `civilsuite.pro.lifetime`, `civilsuite.pro.monthly`, `civilsuite.pro.yearly`.

Nouveautés : 6 langues supplémentaires (italien, néerlandais, catalan, turc, japonais, coréen) ; langue des PDF réglable ; unités SI ou US customary (en US : méthodes ACI et ASTM seulement, valeurs natives en psi et en pouces) ; rapports de chantier par shift jour/nuit avec Heat Index (maximum, heure du pic, moyenne) et fuseau horaire du chantier ; formes de barres BS 8666:2020 ; recouvrement des treillis soudés EN 1992-1-1 §8.7.5 ; partage et import de fichiers `.civilsuite` (Premium) ; aide intégrée avec toutes les formules.

Corrections faites pendant l'audit (bugs trouvés dans la version reçue) :

| # | Problème | Correction |
|---|---|---|
| 1 | Schéma de barre qui chevauchait les champs de saisie (ferraille) | Schéma dans un cadre de hauteur fixe |
| 2 | Légendes et axes des graphiques (granulométrie, montée en résistance) toujours en français | Traduits dans les 20 langues |
| 3 | Titres et sous-titres des modules de l'écran d'accueil absents du catalogue de traduction | Ajoutés (20 langues) |
| 4 | Cases « traces de rouille, épaufrures… » du diagnostic de fissure affichées en français quelle que soit la langue | Corrigé |
| 5 | Version minimale écrite en dur (« 1.1 ») dans l'en-tête des fichiers d'échange | Remplacée par la version réelle de l'app |
| 6 | Données météo : Apple exige, pour toute valeur calculée à partir de ses données, la mention « données modifiées », et le lien vers les sources dans ce qui est affiché | Mention ajoutée sous le Heat Index (app) et dans la note du PDF, avec le lien des sources Apple |
| 7 | Politique de confidentialité inexacte : elle citait l'Apple Watch et une autorisation photothèque que l'app ne demande pas, et ne parlait ni du géocodage, ni des fichiers `.civilsuite`, ni du formulaire de commentaires | Réécrite dans les 14 langues |
| 8 | Mots-clés App Store de 8 langues au-dessus de la limite d'Apple (100 **octets**, pas 100 caractères) | Raccourcis |
| 9 | 3 clés de traduction obsolètes avec des formats incohérents | Supprimées |
| 10 | Fichier de test StoreKit : partage familial différent entre les produits | Harmonisé (n'a d'effet qu'en test local) |

### Limite à connaître

Je n'ai pas de compilateur Swift dans mon environnement. Le code a été relu et contrôlé par des outils de structure, et 20 tests unitaires nouveaux ont été écrits (57 au total), mais **le premier build se fera chez vous**. S'il y a une erreur de compilation, copiez le message complet (clic droit sur l'erreur > « Copy ») et envoyez-le moi : la correction est en général rapide.

---

## Partie A — Préparer et vérifier le projet dans Xcode

### A1. Remplacer le projet

1. Dans le Finder, renommez votre dossier actuel `CivilSuite` en `CivilSuite_avant_1.0b5` (sauvegarde).
2. Copiez le nouveau dossier `CivilSuite` à la même place.
3. Double-cliquez sur `CivilSuite.xcodeproj`. Xcode s'ouvre (le projet a été créé avec Xcode 27.1).

### A2. Vérifier la signature et les capacités

1. Dans la colonne de gauche, cliquez sur l'icône bleue **CivilSuite** (tout en haut).
2. Au centre, sous **TARGETS**, sélectionnez **CivilSuite**.
3. Onglet **Signing & Capabilities** :
   - **Team** : votre équipe (identifiant 66B46V8Q2U dans le projet) ; « Automatically manage signing » coché.
   - **Bundle Identifier** : `com.nicolasmathis.CivilSuite` (ne le changez pas : il est lié à votre fiche App Store Connect).
   - Vous devez voir les blocs **iCloud** (case **iCloud Documents** cochée, conteneur `iCloud.com.nicolasmathis.CivilSuite`), **WeatherKit** et **In-App Purchase**. S'il manque un bloc, cliquez sur **+ Capability** et ajoutez-le.
4. WeatherKit doit aussi être activé côté portail développeur : developer.apple.com > Account > **Certificates, Identifiers & Profiles** > **Identifiers** > `com.nicolasmathis.CivilSuite` : WeatherKit coché dans l'onglet **Capabilities** et dans l'onglet **App Services**. Sans cela, la météo échoue sur les appareils réels.
5. Onglet **General** : **Version** = `1.0`, **Build** = `5`.
   - Si la version 1.0 a déjà été publiée sur l'App Store, mettez `1.1`.
   - Si un build 5 a déjà été envoyé à App Store Connect, mettez `6` (chaque envoi exige un numéro de build plus grand).

### A3. Compiler

1. Menu **Product > Clean Build Folder** (⇧⌘K).
2. En haut, à côté du bouton ▶, choisissez un simulateur, par exemple un iPhone récent.
3. Menu **Product > Build** (⌘B).
4. S'il apparaît des erreurs rouges : ouvrez le navigateur d'erreurs (icône triangle ⚠ dans la colonne de gauche), copiez les messages et envoyez-les moi. Les avertissements jaunes ne bloquent pas la soumission.

### A4. Lancer les tests unitaires

1. Menu **Product > Test** (⌘U).
2. Attendez la fin (quelques minutes). Dans le navigateur de tests (icône losange), tout doit être vert. Un test rouge = envoyez-moi son nom et le message.

### A5. Essai manuel rapide (simulateur iPhone, puis iPad)

Lancez l'app (⌘R) et vérifiez, dans l'ordre :

| Écran | À vérifier |
|---|---|
| Accueil | Les 7 modules, l'icône ⚙ (Réglages) en haut à droite, l'icône bac (calculs, import, aide) en haut à gauche |
| Réglages > Unités | Passer en « US customary » : un message de confirmation apparaît ; les méthodes Eurocode/EN deviennent grisées dans Formulation et Fissures |
| Formulation béton (en US) | Méthode ACI, résultats en psi, lb/yd³ |
| Ferraille | Schéma de barre sans chevauchement ; forme BS 8666 ; recouvrement des treillis (en SI) |
| Rapport de chantier | Un jour > « Nouveau rapport pour ce jour » > Jour ou Nuit. Avec la position autorisée : météo, Heat Index (max, heure, moyenne), mention « données modifiées » |
| PDF | Aperçu du PDF, changement de langue depuis l'aperçu |
| Partage | Dans un module, « Partager » > créer le fichier `.civilsuite` |
| Aide | Icône bac > Aide : formules affichées correctement |

Changer la langue de l'app sans changer celle du simulateur : menu **Product > Scheme > Edit Scheme… > Run > Options > App Language**. Testez au moins une nouvelle langue (italien ou japonais).

Achats en test : lancée depuis Xcode, l'app utilise le fichier `Products.storekit` (achats fictifs, rien n'est facturé). Menu **Debug > StoreKit > Manage Transactions** pour annuler un achat test.

---

## Partie B — Mettre le site à jour (GitHub Pages)

Le site actuel est `https://civilsuite.github.io/civilsuite`. Toutes les anciennes adresses restent valables : le nouveau site ajoute 6 langues et l'aide PDF, il ne supprime rien.

1. Ouvrez github.com, connectez-vous, ouvrez le dépôt qui publie le site (celui dont **Settings > Pages** affiche `civilsuite.github.io/civilsuite`).
2. Cliquez sur **Add file > Upload files**.
3. Ouvrez le dossier `site_a_publier` dans le Finder, sélectionnez **tout son contenu** (pas le dossier lui-même) : `index.html`, `style.css`, `404.html` et les dossiers `privacy`, `terms`, `support`, `help`. Glissez-le dans la page GitHub.
4. En bas, message de validation : « Site 1.0 build 5 : 14 langues + aide PDF », puis **Commit changes**. Les fichiers existants sont remplacés.
5. Onglet **Actions** : attendez que le déploiement passe au vert (souvent une à deux minutes).
6. Vérifiez dans un navigateur (de préférence en navigation privée, pour éviter le cache) :
   - `https://civilsuite.github.io/civilsuite/privacy/fr.html` (date « 24 octobre 2026 »)
   - `https://civilsuite.github.io/civilsuite/privacy/ja.html`
   - `https://civilsuite.github.io/civilsuite/support/it.html`
   - `https://civilsuite.github.io/civilsuite/help/fr.pdf` et `.../help/en.pdf`

L'app ouvre ces pages automatiquement dans sa langue (écran Premium, Réglages, Aide en PDF). Pour l'espagnol, le portugais, l'allemand, etc., l'aide PDF est en anglais (elle n'existe qu'en français et en anglais ; l'aide intégrée à l'app suit la même règle).

---

## Partie C — Archiver et envoyer le build

1. Dans Xcode, à côté du bouton ▶, choisissez la destination **Any iOS Device (arm64)**.
2. Menu **Product > Archive**. La compilation prend quelques minutes ; la fenêtre **Organizer** s'ouvre ensuite sur l'archive.
3. Cliquez sur **Validate App**, laissez les options par défaut, puis **Validate**. Corrigez ce qui est signalé (envoyez-moi le message en cas de doute).
4. Cliquez sur **Distribute App** > **App Store Connect** > **Upload**, options par défaut, puis **Upload**.
5. Question sur le chiffrement : elle ne devrait pas apparaître, car le projet déclare déjà `ITSAppUsesNonExemptEncryption = NO` (l'app n'utilise que le chiffrement fourni par iOS ; l'empreinte SHA-256 des fichiers `.civilsuite` sert au contrôle d'intégrité, pas à chiffrer).
6. Attendez l'e-mail d'Apple indiquant que le build est traité (le délai varie ; je ne sais pas le chiffrer précisément). Le build apparaît ensuite dans App Store Connect, onglet **TestFlight**.

Facultatif mais utile : dans **TestFlight**, ajoutez-vous comme testeur interne et installez le build sur votre iPhone. C'est le seul moyen de tester la vraie météo (WeatherKit) et les achats en environnement sandbox d'Apple.

---

## Partie D — App Store Connect

Ouvrez appstoreconnect.apple.com > **Apps** > **CivilSuite**.

### D1. Vérifications de compte (une seule fois)

1. **Business** (en haut) > onglet **Agreements** : le contrat **Paid Apps** doit être actif, avec coordonnées bancaires et fiscales complètes. Sans lui, les achats intégrés ne peuvent pas être vendus.
2. Même page, section **Compliance** > **Digital Services Act** > **Complete Compliance Requirements** : déclarez votre statut de commerçant (« trader »). Apple ne peut pas le déterminer à votre place. Si vous vendez des abonnements, vous êtes très probablement un commerçant au sens du DSA ; dans ce cas, votre adresse, votre téléphone et votre e-mail seront affichés sur la fiche dans l'Union européenne. Je ne peux pas trancher ce point juridique pour vous.

### D2. Ajouter les 6 langues et remplir « App Information »

1. Barre latérale > **App Information**.
2. En haut à droite, cliquez sur la langue principale pour ouvrir le menu des langues ; dans **Not Localized**, survolez la langue et cliquez sur **(+)**. Faites-le pour : **Catalan**, **Dutch**, **Italian**, **Japanese**, **Korean**, **Turkish**.
3. Pour chacune des 20 langues (choisies une par une dans ce même menu), renseignez :
   - **Name** : `CivilSuite`
   - **Subtitle** : colonne « Sous-titre » du classeur, onglet « Fiche App Store »
   - **Privacy Policy URL** : selon le tableau ci-dessous
   - puis **Save**.

| Langue App Store Connect | Privacy Policy URL |
|---|---|
| English (U.S., U.K., Australia, Canada) | `https://civilsuite.github.io/civilsuite/privacy/en.html` |
| French, French (Canada) | `…/privacy/fr.html` |
| Spanish (Spain), Spanish (Mexico) | `…/privacy/es.html` |
| Portuguese (Brazil), Portuguese (Portugal) | `…/privacy/pt.html` |
| German | `…/privacy/de.html` |
| Italian | `…/privacy/it.html` |
| Dutch | `…/privacy/nl.html` |
| Catalan | `…/privacy/ca.html` |
| Russian | `…/privacy/ru.html` |
| Turkish | `…/privacy/tr.html` |
| Chinese (Traditional) | `…/privacy/zh.html` |
| Japanese | `…/privacy/ja.html` |
| Korean | `…/privacy/ko.html` |
| Hindi | `…/privacy/hi.html` |

4. Toujours dans **App Information** :
   - **Category** : Productivity (cohérent avec le projet), secondaire au choix (Utilities ou Business).
   - **Content Rights** : l'app affiche des données Apple Weather, fournies sous licence via WeatherKit. Répondez que l'app affiche du contenu tiers et que vous disposez des droits nécessaires. Je ne sais pas si Apple considère les données WeatherKit comme du « contenu tiers » pour cette question : la réponse ci-dessus est la plus prudente.
   - **Age Rating** > **Edit** : répondez **Non / None** partout (contrôles parentaux, vérification d'âge, accès web libre, contenu généré diffusé, messagerie, publicité, thèmes matures, médical, sexualité, violence, jeux de hasard). Résultat attendu : **4+**. Justification : l'aide est une page locale et les liens externes s'ouvrent dans Safari ; les fichiers `.civilsuite` s'échangent de personne à personne par la feuille de partage d'iOS, ce n'est ni un réseau social ni une messagerie. Je ne sais pas si Apple rattache le Heat Index à « Health or Wellness Topics » ; à mon avis non, c'est un indice météo.
   - **Digital Services Act** (rubrique App Store Regulations and Permits) : vérifiez que le statut déclaré en D1 apparaît.

### D3. Confidentialité de l'app (App Privacy) — je confirme : « Data Not Collected »

Barre latérale > **App Privacy**.

1. **Data Collection** : si la réponse actuelle est déjà « No, we do not collect data from this app », **ne changez rien**. Sinon : **Get Started** / **Edit** > **No, we do not collect data from this app** > **Save**, puis **Publish**.
2. Ce qui change réellement, c'est le **texte** de la politique (site, partie B) et les **URL des 6 nouvelles langues** (D2).

Pourquoi c'est juste, point par point (vérifié dans le code du build 5 et sur la page Apple « App privacy details ») :

| Donnée | Ce que fait l'app | Conclusion |
|---|---|---|
| Calculs, rapports, photos | Restent sur l'appareil ou dans l'iCloud privé de l'utilisateur | Apple : « Data that is processed only on device is not "collected" » |
| Position du chantier | Envoyée à Apple Weather (WeatherKit) et au géocodage Apple Plans (MapKit) | Apple : « You are not responsible for disclosing data collected by Apple » |
| Achats | Vérifiés par StoreKit sur l'appareil | Rien ne vous parvient |
| Fichiers `.civilsuite`, PDF | Partagés par l'utilisateur, au destinataire qu'il choisit | Rien ne transite par vous |
| Formulaire de commentaires | Prépare un e-mail que l'utilisateur envoie lui-même avec Mail | L'app ne transmet rien. Je ne sais pas si Apple qualifierait cet e-mail de collecte ; sa page ne traite pas ce cas |
| Pistage, publicité, analyse | Aucun SDK tiers ; `NSPrivacyTracking = false` | Pas d'App Tracking Transparency |

Attention pour l'avenir : si vous activez l'envoi direct des commentaires (identifiant Formspree dans `FeedbackService.swift`), l'app transmettra elle-même e-mail, message et informations techniques à un tiers. Il faudra alors déclarer **Contact Info > Email Address**, **User Content > Customer Support** et **Diagnostics > Other Diagnostic Data** (finalité App Functionality, liées à l'utilisateur, sans pistage) et mettre à jour la politique.

### D4. Prix et disponibilité

Barre latérale > **Pricing and Availability** : prix de l'app **Free (0)** ; pays : tous, ou à votre choix.

### D5. Abonnements et achat à vie

Les produits existent déjà : on ajoute les langues et on vérifie.

**Groupe d'abonnements**

1. Barre latérale > **Monetization** > **Subscriptions** > votre groupe.
2. Section **App Store Localization** du groupe : ajoutez les 6 langues avec le **même Subscription Group Display Name** que celui déjà saisi pour les 14 autres (le dossier précédent proposait `CivilSuite Premium` ; le fichier de test `Products.storekit` dit « CivilSuite Pro » mais n'est pas lu par l'App Store).

**Chaque abonnement (mensuel, puis annuel)**

1. Cliquez sur l'abonnement.
2. **Subscription Prices** : 0,99 € (mensuel), 9,99 € (annuel) pour le pays de référence ; App Store Connect propose les équivalents dans les autres pays.
3. **Introductory Offers** (même zone des prix) : offre **Free**, durée **3 days**, pour tous les pays. Vérifiez qu'elle existe pour les deux abonnements.
4. **App Store Localization** : ajoutez les 6 langues ; **Display Name** et **Description** = onglet « Achats intégrés » du classeur (lignes vertes). Vérifiez les 14 autres langues.
5. **Review Information** : une capture de l'écran Premium de l'app (prise dans le simulateur) et, en note : « Unlocks all modules and .civilsuite file sharing. »
6. Statut attendu : **Ready to Submit**.

**Achat à vie**

1. **Monetization** > **In-App Purchases** > `civilsuite.pro.lifetime`.
2. Type : Non-Consumable ; prix 12,99 € ; localisations des 6 langues (même onglet du classeur) ; capture de revue ; statut **Ready to Submit**.
3. **Family Sharing** : choisissez le même réglage pour les trois produits. Une fois activé dans App Store Connect, il ne peut plus être désactivé.

Règle d'Apple à connaître : le premier achat intégré de chaque type (non consommable, abonnement auto-renouvelable) doit être soumis **avec une version de l'app**, et un nouvel abonnement avec son groupe, dans la même soumission (voir D7).

### D6. La fiche de la version (iOS App 1.0)

Barre latérale > **iOS App** > **1.0 Prepare for Submission**.

1. **Langues** : comme en D2, menu des langues en haut à droite, **(+)** pour les 6 nouvelles langues (si vous l'avez déjà fait en D2, elles y sont déjà : ajouter une langue d'un côté l'ajoute de l'autre).
2. Pour **chacune des 20 langues**, copiez depuis l'onglet « Fiche App Store » du classeur (une ligne par langue, les 6 nouvelles en vert) :
   - **Promotional Text**, **Description**, **Keywords**, **Support URL**, **Marketing URL**.
   - **What's New in This Version** : ce champ n'existe pas pour la toute première version d'une app ; il n'apparaît que si la 1.0 est déjà publiée (vous soumettez alors une 1.1 : copiez la colonne « Nouveautés »).
   - Les 14 langues existantes ont un texte **mis à jour** (nouvelles fonctions) : remplacez l'ancien.
3. **Captures d'écran** (exigences Apple vérifiées le 24/10/2026) :
   - iPhone : au moins une capture « Dynamic Island (medium) » : **1179 × 2556** ou **1206 × 2622** pixels (portrait).
   - iPad : au moins une capture iPad 13 pouces : **2064 × 2752** ou **2048 × 2732** pixels.
   - 1 à 10 captures par taille, en .png ou .jpg, sans transparence. Les autres tailles sont dérivées automatiquement.
   - Comment faire : lancez l'app dans un simulateur, **File > Save Screen** (⌘S) dans le Simulateur ; vérifiez la taille en pixels (Finder > Lire les informations). Si elle ne correspond pas exactement, essayez un autre modèle de simulateur.
   - Suggestion de 5 captures : accueil ; formulation béton avec résultats ; analyse de fissure ; rapport de chantier avec météo et Heat Index ; aperçu PDF ou réglage des unités.
   - Les langues sans captures propres reprennent celles de la langue principale : c'est accepté. Des captures dans chaque langue sont un plus, pas une obligation.
4. **Build** : cliquez sur **Add Build** (ou **+**) et choisissez le build **5** (ou celui que vous avez envoyé).
5. **In-App Purchases and Subscriptions** (section de la page de version, présente tant que les premiers achats intégrés ne sont pas approuvés) : cochez les trois produits. Si la section n'apparaît pas, passez par D7 étape 2.
6. **App Review Information** : décochez **Sign-in required** ; contact : Nicolas Mathis, +34 653 135 500, civilsuite.app@proton.me ; **Notes** : collez l'onglet « Infos revue Apple » (texte anglais). Pièce jointe : inutile.
7. **Version Release** : « Manually release this version » si vous voulez choisir le jour de la sortie ; sinon « Automatically ».
8. **Copyright** : `2026 Nicolas Mathis`.
9. **Save** en haut à droite.

### D7. Soumettre

1. En haut de la page de version : **Add for Review**.
2. Dans la fenêtre de soumission, vérifiez qu'elle contient : la version 1.0, le groupe d'abonnements, les deux abonnements et l'achat à vie. S'il manque un produit : **Monetization** > le produit > **Add for Review** > ajoutez-le à la soumission existante.
3. **Submit for Review**.
4. Statut « Waiting for Review », puis « In Review ». Apple répond dans **App Review** (barre latérale) : répondez en anglais, factuellement, en citant les notes de revue.

### D8. Après l'approbation

- Si vous avez choisi la sortie manuelle : **Release This Version**.
- Vérifiez la fiche sur l'App Store dans 2 ou 3 langues, testez un achat réel (vous pouvez vous rembourser via reportaproblem.apple.com si besoin).
- Gardez ce dossier : `build.py` régénère tous les textes si vous changez une phrase.

---

## Annexes

### Les motifs de refus les plus probables et la parade déjà en place

| Règle Apple (App Review Guidelines) | Risque | Ce qui est déjà fait |
|---|---|---|
| 3.1.2 Abonnements | Écran d'achat sans durée, prix, liens légaux | L'écran Premium affiche titre, durée, prix, essai, « Restaurer les achats », politique de confidentialité et conditions (EULA Apple) |
| 5.1.1 Collecte de données | Politique qui ne décrit pas le comportement réel | Politique réécrite d'après le code (position, géocodage, partage, e-mail) |
| 2.1 Complétude | Fonction qui ne marche pas pendant la revue (météo) | Le rapport fonctionne sans position ; les notes expliquent comment tester |
| WeatherKit (contrat) | Marque et lien des sources absents, valeurs dérivées non signalées | Logo Apple Weather et lien « Sources des données » dans l'app ; mention et lien dans le PDF ; mention « données modifiées » |
| 2.3 Métadonnées | Mots-clés trop longs, captures non conformes | Mots-clés ≤ 100 octets dans les 20 langues ; tailles de captures ci-dessus |

### À faire relire si possible

- Les traductions des 6 nouvelles langues (app, fiche, site) ont été rédigées par moi sans relecture par un locuteur natif. Une relecture, même rapide, par un collègue italien, néerlandais, catalan, turc, japonais ou coréen est recommandée avant la publication de la fiche, au moins pour la description.
- Les textes juridiques sont des textes types adaptés au fonctionnement réel de l'app, pas un avis d'avocat.

### Points ouverts, non tranchés ici

- Le statut DSA (commerçant ou non) : décision juridique qui vous appartient.
- La cible UITests (`CivilSuiteUITests`) est vide : sans effet sur la soumission.
- Apple prévoit des captures « iPhone Duo » obligatoires à partir d'avril 2027 pour les apps compilées avec le SDK iOS 27.1 ou plus récent (page des spécifications de captures) : rien à faire pour cette soumission.

### Sources (consultées le 24/10/2026)

- Apple, App privacy details on the App Store — https://developer.apple.com/support/app-privacy-on-the-app-store/
- Apple, Managing metadata in your app by using locale shortcodes — https://developer.apple.com/documentation/appstoreconnectapi/managing-metadata-in-your-app-by-using-locale-shortcodes
- App Store Connect Help, Localize app information — https://developer.apple.com/help/app-store-connect/manage-app-information/localize-app-information
- App Store Connect Help, Platform version information (mots-clés 100 octets, « What's New » absent en 1re version) — https://developer.apple.com/help/app-store-connect/reference/app-information/platform-version-information/
- App Store Connect Help, App information — https://developer.apple.com/help/app-store-connect/reference/app-information/app-information
- App Store Connect Help, Screenshot specifications — https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications/
- App Store Connect Help, Submit an In-App Purchase — https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/submit-an-in-app-purchase
- App Store Connect Help, Age ratings values and definitions — https://developer.apple.com/help/app-store-connect/reference/app-information/age-ratings-values-and-definitions/
- App Store Connect Help, Manage EU Digital Services Act compliance information — https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-eu-digital-services-act-compliance-information
- Apple, Get started with WeatherKit (attribution, valeurs dérivées) — https://developer.apple.com/weatherkit/get-started/
