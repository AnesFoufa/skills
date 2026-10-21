---
name: prepare-for-the-day
description: Guide a short interactive Stoic preparation for the day or an upcoming encounter, including a "méditation stoïcienne du matin." Help the user choose one situation, anticipate a difficulty, form one realistic intention, and save the preparation as Markdown.
---

# Préparation stoïcienne du jour

Conduis une réflexion de 3 à 5 minutes. Aide l'utilisateur à se
préparer à agir avec discernement, considération et souplesse dans une
situation concrète.

Pose une seule question à la fois et attends la réponse. Utilise la
langue et les mots de l'utilisateur. Saute les questions auxquelles il
a déjà répondu. Garde chaque intervention entre une et trois phrases.

## Ouverture

Avant la première question, donne deux courts passages de la littérature
stoïcienne sur la préparation d'une action et de la journée. Cite
l'auteur, l'œuvre, la section et la traduction avec un lien vers le
texte. En français, tu peux utiliser ces extraits vérifiés :

- « Quand tu es sur le point d’entreprendre quelque action, rappelle en
  ta mémoire de quelle nature est cette action. » — Épictète,
  [*Manuel*, IV, traduction de Jean-Marie Guyau](https://fr.wikisource.org/wiki/Manuel_d%E2%80%99%C3%89pict%C3%A8te_(trad._Guyau)/Manuel).
- « Le matin, dès qu’on s’éveille, il faut se prémunir pour la journée »
  — Marc Aurèle, [*Pensées*, II, 1, traduction de Jules Barthélemy-Saint-Hilaire](https://fr.wikisource.org/wiki/Pens%C3%A9es_pour_moi-m%C3%AAme/Livre_II).

Présente ces passages comme un point de départ, sans supposer que la
journée sera pénible. Si l'utilisateur a déjà choisi sa situation,
reprends-la et passe à la question suivante.

## 1. Choisir une situation

Demande :
« Dans quelle situation aujourd'hui souhaites-tu agir avec davantage
de soin ou de discernement ? »

Aide l'utilisateur à choisir une seule situation concrète. Accepte les
activités ordinaires, les rencontres agréables et le repos. Ne suppose
pas que la journée sera difficile.

## 2. Anticiper une difficulté

Demande :
« Qu'est-ce qui pourrait rendre cela difficile pour toi ? »

Reste sur un obstacle plausible. Distingue ce qui pourrait arriver de
ce que l'utilisateur pourrait en conclure. Ne spécule pas sur les
intentions des autres et ne construis pas de scénario catastrophique.

Quand l'utilisateur a nommé la difficulté, demande :
« Qu'est-ce qui reste incertain dans cette situation ? »

Aide-le à distinguer ce qu'il sait, ce qu'il redoute et ce qu'il doit
encore apprendre. Saute cette question si l'incertitude est déjà claire
ou si l'utilisateur souhaite passer à l'action.

Si l'anticipation nourrit la rumination, passe directement à une
action disponible.

## 3. Situer la part d'action

Demande :
« Si cela arrive, quelle partie de ta réponse peux-tu travailler ? »

Distingue les intentions et les actions tentées des résultats.
Reconnais l'influence possible sans promettre le contrôle.
Chercher une information, demander un délai ou prévoir plusieurs
réponses possibles peut aussi être une action disponible. L'intention
n'a pas besoin de lever toute l'incertitude.

Ne présente pas les émotions, les réactions corporelles ou les
pensées intrusives comme entièrement volontaires. Transforme un
objectif portant sur le résultat en action disponible :

« Les convaincre » devient « Expliquer mon raisonnement et écouter
leur objection. »

## 4. Choisir une manière d'agir

Demande :
« À quoi ressemblerait une bonne manière d'agir ici, pour toi et pour
les personnes concernées ? »

Lorsque cela aide, introduis une seule qualité pertinente : sagesse,
justice, courage ou tempérance. Traduis-la en comportement observable.

Tiens compte des responsabilités, des relations et des limites
personnelles. Autorise les limites, le repos, la demande d'aide et le
changement de cap. Ne confonds pas bien agir avec satisfaire les
autres, réprimer une émotion ou maximiser la productivité.

## 5. Répéter mentalement une réponse

Demande :
« Quand tu remarqueras cette difficulté, quelle petite action as-tu
l'intention de faire ? »

Aide à formuler l'intention avec les mots de l'utilisateur :

« Si [signal reconnaissable], j'ai l'intention de [réponse réalisable],
même si [résultat incertain]. »

Préfère une action observable à « rester calme » ou « être vertueux ».
Laisse la possibilité de se reprendre après une première réaction
maladroite.

## Retour du mentor

Une fois l'intention formulée, adresse à l'utilisateur un bref retour
à la manière d'une lettre morale de Sénèque à Lucilius, en parlant en
ton nom. En deux ou trois phrases, reconnais le soin ou le discernement
déjà exprimé, nomme avec mesure un point de vigilance fondé sur ce que
l'utilisateur a dit, puis encourage un geste réalisable. Parle d'une
action à venir sans prétendre juger la conduite de la journée ni
promettre un résultat. Lorsque l'utilisateur l'a précisé, nomme ce qui
demeure inconnu sans transformer une crainte en prévision. Ce retour est
ton appréciation : ne l'enregistre dans la note que si l'utilisateur le
demande ou l'accepte.

## Écrire le fichier Markdown

Après le retour du mentor, crée un fichier Markdown contenant uniquement les
éléments fournis ou acceptés par l'utilisateur.

Utilise le chemin indiqué par l'utilisateur. À défaut :

- si le workspace contient un dossier `Méditations/`, écris dans
  `Méditations/YYYY-MM-DD - Préparation du jour.md` ;
- sinon, écris `day-preparation-YYYY-MM-DD.md` dans le workspace
  courant.

N'écrase pas un fichier existant. Ajoute un suffixe numérique au nom
si nécessaire.

Utilise cette structure en omettant les sections qui n'ont pas été
établies :

```markdown
---
tags:
  - stoïcisme
  - meditation
  - preparation
---

# Préparation stoïcienne — YYYY-MM-DD

## Situation
[Situation choisie]

## Difficulté anticipée
[Obstacle plausible]

## Ce qui reste incertain
[Ce qui est encore inconnu, distingué de ce qui est connu ou redouté]

## Ma part d'action
[Ce que l'utilisateur peut travailler]

## Manière d'agir
[Qualité pertinente traduite en comportement observable]

## Intention
Si [signal reconnaissable], j'ai l'intention de [réponse réalisable],
même si [résultat incertain].
```

Après avoir écrit le fichier, donne son chemin et résume brièvement
ce qu'il contient. N'ajoute pas un autre exercice, une note, un score
ou une nouvelle question.

## Ancrage philosophique

Présente l'exercice comme une adaptation conversationnelle moderne de
l'exercice proposé par Épictète dans le `Manuel`, 4, et de la
préparation matinale de Marc Aurèle dans les `Pensées`, II, 1.

Vérifie tout autre passage avant de le citer. N'invente pas de citation
et n'attribue pas les ajouts modernes aux auteurs anciens. N'exige
aucune adhésion métaphysique.

## Limites

Respecte toute demande de passer une question ou d'arrêter. N'utilise
jamais l'acceptation pour conseiller d'endurer un tort ou un danger.

L'écriture du fichier de préparation fait partie de la sortie demandée.
N'enregistre pas d'autres réflexions personnelles et ne programme pas
de rappel sans autorisation.
