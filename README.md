# 🎛️ VOLTAGE.16

Séquenceur pour navigateur web, combinant une ligne de basse de
type acid et cinq voix de percussion. L'ensemble du son est synthétisé
en temps réel via la Web Audio API — aucun échantillon audio, aucune
dépendance externe, aucun serveur requis.

Ce projet est une implémentation originale, pensée dans le même esprit
que les boîtes à rythmes/synthétiseurs analogiques classiques
(basse acide + séquenceur de percussions), mais entièrement réécrite
à partir de zéro : aucun code ni asset n'a été repris d'un logiciel
tiers.

---

## Fonctionnalités

**Synthèse**
- Ligne de basse monophonique — dent de scie/carrée, filtre passe-bas
  avec enveloppe, accent et glissando (slide) par pas.
- Voix lead — deuxième synthé monophonique (triangle/sinus/dent de
  scie), mêmes gestes que la basse.
- Cinq percussions synthétisées en direct : grosse caisse, caisse
  claire, charley fermé, charley ouvert, clap.
- Delay synchronisé au tempo, avec réglages de mix et de feedback.

**Séquenceur**
- 8, 16 ou 32 pas, au choix.
- Quatre banques de motifs (A/B/C/D) indépendantes, changement à la
  volée, même pendant la lecture.
- Swing réglable, tempo de 60 à 180 BPM.
- Mute et solo par piste (basse, lead, chaque percussion).
- Annuler / Rétablir sur chaque modification.
- Générateur de motif aléatoire pour la basse (gamme pentatonique
  mineure).

**Sauvegarde et partage**
- Sauvegarde automatique locale (le navigateur retient l'état entre
  deux visites).
- Export / import du projet complet en fichier JSON.
- Génération d'un lien partageable contenant le motif actuel.

**Export audio**
- Enregistrement en direct de ce qui est joué (`.webm`).
- Rendu hors ligne propre d'une boucle en fichier `.wav`.

**Autres**
- Sortie MIDI (Web MIDI API) vers un périphérique ou port virtuel,
  basse/lead/percussions sur des canaux distincts.
- Contrôles façon matériel analogique (boutons rotatifs actionnés par
  glissement vertical).
- Compatible tactile, focus clavier visible, respecte
  `prefers-reduced-motion`.

---

## Utilisation

Aucune installation n'est nécessaire.

1. Ouvrir `voltage16.html` directement dans le navigateur (double-clic).
2. Cliquer sur **Lecture** pour démarrer le motif par défaut.
3. Construire un motif :
   - clic sur un pas : activer/désactiver la note ou le coup de
     percussion ;
   - glisser un pas de basse ou de lead verticalement : changer la
     hauteur de la note ;
   - boutons **ACC** / **SLD** sous chaque pas mélodique : accent et
     glissando ;
   - boutons **M** / **S** à côté de chaque piste : couper ou isoler
     cette piste ;
   - boutons rotatifs : cliquer-glisser verticalement pour régler la
     valeur.
4. Utiliser la barre d'outils pour changer de banque, ajuster le
   nombre de pas, annuler une modification, générer une basse
   aléatoire, sauvegarder ou exporter.

---

## 📸 Screenshot

![Dashboard](screenshot.png)

---
## Barre d'outils

| Bouton                | Effet                                                          |
|------------------------|-------------------------------------------------------------------|
| **Pas** (8/16/32)       | Change la longueur du motif ; les notes existantes sont conservées |
| **Banque** (A–D)        | Bascule vers un autre motif indépendant                           |
| **Annuler / Rétablir**  | Historique des modifications du motif affiché                     |
| **🎲 Basse aléatoire**  | Génère un nouveau motif de basse                                  |
| **Exporter JSON**       | Télécharge l'état complet (banques, réglages, mute/solo)          |
| **Importer JSON**       | Recharge un état exporté précédemment                             |
| **🔗 Copier le lien**   | Copie une URL contenant le motif actuel                           |
| **Exporter WAV**        | Rend une boucle propre en fichier audio téléchargeable            |

---
##  🔴 Enregistrement vs export WAV

Deux façons différentes de capturer le son, pour deux usages
différents :

- **Enregistrer** (bouton rouge, en haut) capture en direct exactement
  ce qui sort des haut-parleurs pendant que ça joue — y compris le
  delay et les réglages ajustés en direct. Format `.webm`.
- **Exporter WAV** (dans la barre d'outils) rend une seule boucle hors
  ligne, sans delay, en `.wav` non compressé — pratique pour réimporter
  le son ailleurs (montage, sampler, etc.).

---

## Sortie MIDI

Le module **Sortie MIDI**, en bas de page, permet d'envoyer les notes
jouées vers un périphérique MIDI (matériel ou port virtuel) :

1. Cocher **Activer l'envoi MIDI**.
2. Choisir le port de sortie dans la liste déroulante.
3. Lancer la lecture — la basse part sur le canal 1, le lead sur le
   canal 2, les percussions sur le canal 10 (convention General MIDI :
   grosse caisse = 36, caisse claire = 38, charley fermé = 42, charley
   ouvert = 46, clap = 39).

Nécessite un navigateur compatible Web MIDI (Chrome, Edge).

---

## Architecture technique

| Élément                 | Détail                                                                                            |
|-------------------------|---------------------------------------------------------------------------------------------------|
| 🐝 Basse                | Oscillateur → filtre passe-bas (cutoff, résonance, enveloppe) → gain                              |
| 🥁 Grosse caisse        | Oscillateur sinusoïdal, glissando de fréquence 150 Hz → 40 Hz                                     |
| 🎯 Caisse claire        | Bruit filtré (passe-bande) + tonalité courte                                                      |
| 🔨 Charleys             | Bruit filtré (passe-haut), durée d'extinction courte ou longue                                    |
| 👏 Clap                 | Trois impulsions de bruit filtré légèrement décalées                                              |
| 🔨 Cadencement          | Scheduler à anticipation (lookahead), synchronisé sur l'horloge audio, pas sur `setInterval` seul |
| 🔴  Enregistrement      | `MediaStreamAudioDestinationNode` + `MediaRecorder`                                               |
| Sortie MIDI              | Web MIDI API (`navigator.requestMIDIAccess`)                        |
| Sauvegarde               | `localStorage` (auto) + export/import JSON + lien encodé en base64  |

Fichier unique (`voltage16.html`), sans framework ni étape de build.
Les instruments (basse, lead, percussions) sont écrits comme des
fonctions pures prenant un contexte audio et une destination en
paramètres — le même code sert à la lecture en direct et au rendu
hors ligne pour l'export WAV.

---

## Limitations connues

- Basse et lead sont chacun monophoniques (une seule note à la fois
  par voix).
- Le démarrage audio nécessite une interaction utilisateur explicite
  (contrainte standard des navigateurs).
- L'export WAV ne inclut pas le delay (rendu sec, une seule boucle) ;
  utiliser Enregistrer si le delay doit être capturé.
- Le lien partageable n'encode que la banque actuellement affichée,
  pas les quatre banques — utiliser l'export JSON pour tout
  sauvegarder.
- La sortie MIDI dépend du support Web MIDI du navigateur (Chrome,
  Edge ; pas Safari).
---
## Licence
MIT – fais-en ce que tu veux. Transforme-le, réaménage-le, tout est permis.
🎉
