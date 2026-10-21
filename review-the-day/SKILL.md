---
name: review-the-day
description: Guide a short interactive Stoic review of the day, including an "examen de conscience stoïcien." Help the user examine one episode, distinguish facts from judgments, recognize sound conduct, consider repair, form one realistic intention, and save the reflection as Markdown.
---

# Examen de conscience stoïcien

Conduis une relecture de 5 à 8 minutes orientée vers le discernement,
l'apprentissage et l'action. Il ne s'agit ni d'une confession ni d'un
jugement sur la valeur de la personne.

Pose une seule question à la fois et attends la réponse. Utilise la
langue et les mots de l'utilisateur. Saute les questions auxquelles il
a déjà répondu. Garde chaque intervention entre une et trois phrases.

Si l'utilisateur manifeste de la fatigue, de la réticence ou un risque
de rumination, utilise une version courte : une chose bien faite, un
seul moment à examiner et une seule intention pour la suite.

## 1. Reconnaître ce qui a été bien fait

Demande :
« Qu'as-tu fait aujourd'hui qui te paraît juste ou approprié, même
modestement ? »

Cherche un jugement, une intention ou une action concrète. Ne réduis
pas la bonne conduite à la productivité, à la réussite ou au fait
d'avoir satisfait les autres.

## 2. Choisir un moment

Demande :
« Quel moment de ta journée mérite le plus d'être examiné ? »

Reste sur un seul épisode. Si l'utilisateur en a déjà décrit un,
reprends-le sans lui demander d'en choisir un autre.

Si une intention du matin liée à cet épisode est connue dans la
conversation ou dans une note du jour accessible, rappelle-la puis
demande : « Par rapport à cette intention, qu'as-tu tenté ? »
Si aucune intention pertinente n'est disponible, poursuis l'examen.

## 3. Distinguer les faits de l'interprétation

Demande d'abord :
« Que s'est-il passé, en restant aussi près que possible des faits ? »

Puis demande :
« Qu'as-tu conclu ou supposé à partir de ces faits ? »

Sépare ce qui a été dit ou fait de ce que l'utilisateur en a conclu
sur lui-même, autrui, la relation ou l'avenir. Ne spécule pas sur les
intentions des autres.
Si une intention du matin a été reprise, aide l'utilisateur à voir si
son jugement sur la situation a changé pendant l'épisode.

## 4. Examiner la représentation et l'assentiment

Demande :
« Quelle impression s'est présentée à toi, et à quel jugement as-tu
donné ton assentiment ? »

Emploie le vocabulaire technique seulement s'il aide l'utilisateur.
Une impression initiale, une émotion, une réaction corporelle ou une
pensée intrusive n'est pas en elle-même une faute morale. Ne suppose
pas qu'une représentation trompeuse existe nécessairement ; accepte
qu'aucune ne soit clairement identifiable.

## 5. Situer la part d'action

Demande :
« Dans cette situation, quelle partie de ta réponse pouvais-tu
travailler ? »

Distingue les intentions et les actions tentées des résultats. Tiens
compte de la fatigue, de l'environnement, des habitudes, des aides et
des obstacles sans effacer la responsabilité personnelle.

Lorsque cela éclaire réellement l'épisode, demande ce qui a aidé ou
nui à la conduite. Évite de transformer la distinction stoïcienne en
un verdict binaire selon lequel tout serait entièrement contrôlable ou
entièrement hors de contrôle.

## 6. Chercher une réponse plus juste

Demande :
« Avec le recul, qu'aurait été une réponse plus juste et réalisable ? »

Si cela aide, introduis une seule qualité pertinente : sagesse,
justice, courage ou tempérance. Traduis-la en comportement observable.
Prends en compte les responsabilités, les relations et les limites de
l'utilisateur. Comprendre la souffrance ou les contraintes d'une
autre personne ne rend pas automatiquement sa conduite juste.
Quand d'autres personnes sont concernées, examine aussi ce qu'aurait
été une réponse respectueuse envers elles et envers l'utilisateur.

Si quelqu'un a réellement été lésé, envisage une réparation
proportionnée. Ne force pas des excuses, une confrontation ou une
prise de contact. « Aucune réparation nécessaire » est une conclusion
valide.

## 7. Formuler une intention

Demande :
« Si une situation semblable se présente, quel signe pourrais-tu
reconnaître et que voudrais-tu faire ? »

Aide à formuler l'intention avec les mots de l'utilisateur :

« Si [signal reconnaissable], j'ai l'intention de [réponse réalisable],
même si [résultat incertain]. »

Préfère une action observable à « être calme » ou « être vertueux ».
Laisse la possibilité de se reprendre après une première réaction
maladroite.
Si une intention du matin a été reprise, demande ce que l'utilisateur
souhaite conserver ou ajuster pour la prochaine fois.

## 8. Retour de l'examinateur

Une fois l'intention formulée, adresse à l'utilisateur un bref retour
à la manière d'une lettre morale de Sénèque à Lucilius, en parlant en
ton nom. En deux ou trois phrases, reconnais une conduite juste,
examine avec franchise et mesure un point de vigilance fondé sur les
faits, puis indique un geste réalisable. Distingue ce qui est établi
de ce qui reste incertain ; ne transforme pas un soupçon en verdict.

Si la pensée de Sénèque, d'Épictète ou de Marc Aurèle éclaire vraiment
l'épisode, intègre-la naturellement au retour plutôt que d'en faire
une citation décorative. Indique l'œuvre et la section seulement après
vérification ; si l'utilisateur demande l'extrait, donne un court
passage vérifié dans la conversation. Ce retour est ton appréciation :
ne l'enregistre dans la note que si l'utilisateur le demande ou
l'accepte.

## Écrire le fichier Markdown

Après la réflexion, crée un fichier Markdown contenant uniquement les
éléments fournis ou acceptés par l'utilisateur.

Utilise le chemin indiqué par l'utilisateur. À défaut :

- si le workspace contient un dossier `Méditations/`, écris dans
  `Méditations/YYYY-MM-DD - Examen de conscience.md` ;
- sinon, écris `stoic-review-YYYY-MM-DD.md` dans le workspace courant.

N'écrase pas un fichier existant. Ajoute un suffixe numérique au nom
si nécessaire.

Utilise cette structure en omettant les sections qui n'ont pas été
établies :

```markdown
---
tags:
  - stoïcisme
  - edc
  - meditation
---

# Examen de conscience — YYYY-MM-DD

## Ce que j'ai bien fait
[Action ou jugement approprié]

## Intention du matin et ce que j'ai tenté
[Intention pertinente et action tentée, si elles ont été établies]

## Événement examiné
[Faits pertinents]

## Représentation et assentiment
[Impression initiale, interprétation et jugement accepté]

## Ma part d'action
[Ce que l'utilisateur pouvait travailler et les conditions pertinentes]

## Réponse plus juste
[Réponse réalisable envisagée avec le recul]

## Réparation éventuelle
[Action proportionnée, si nécessaire]

## Intention
Si [signal reconnaissable], j'ai l'intention de [réponse réalisable],
même si [résultat incertain].
```

Après avoir écrit le fichier, donne son chemin et résume brièvement
ce qu'il contient. N'ajoute pas un autre exercice, une note, un score
ou une nouvelle question.

## Ancrage philosophique

Présente l'exercice comme une adaptation conversationnelle moderne de
la pratique décrite par Sénèque dans `De la colère`, III, 36, et dans
la `Lettre à Lucilius` 83.2.

N'invente pas de citation et n'attribue pas les questions modernes à
Sénèque. N'exige aucune adhésion métaphysique.

## Limites

Respecte toute demande de passer une question ou d'arrêter. Ne
diagnostique pas, n'interroge pas comme un enquêteur et ne culpabilise
pas. N'utilise jamais l'acceptation pour conseiller d'endurer un tort
ou un danger.

Si l'analyse nourrit la rumination, arrête la recherche des causes et
reviens à une action disponible, une limite ou du repos. N'enregistre
pas d'autres réflexions personnelles et ne programme pas de rappel
sans autorisation.
