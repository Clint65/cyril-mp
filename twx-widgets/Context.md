# Contexte

Construire une skill qui permet d'aider à la création et à la modification de Widgets Thingworx.
Cette skill fait partie du marketplace cyril-mp définie dans le folder @~/Documents/01_Sources/02_IA/mes-skills/
Le folder de la skill est @~/Documents/01_Sources/02_IA/mes-skills/twx-widgets

# Références:

- documentation officielle Thingworx: https://support.ptc.com/help/thingworx/platform/r9.7/en/#page/ThingWorx/Welcome.html
- documentation spécifique aux widgets: https://support.ptc.com/help/thingworx/platform/r9.7/en/#page/ThingWorx/Help/Best_Practices_for_Developing_Applications/visualization_widgets.html#

**Attention**
Je n'utilise pas la méthode standard de thingworx pour créer les widgets: au lieu d'utiliser un script ant pour packager, j'ai construit une structure de projets basée sur Webpack.
Les widgets peuvent être écrite en TypeScript / sass (méthode préférée) grace a la librairie: https://github.com/thingworx-field-work/ThingworxWidgetSupportPackage
ou en javascript / css.

## Projets exemple à scanner pour comprendre la structure, et le packaging d'un projet widget:

Les exemples à scanner sont dans le folder @~/Documents/01_Sources/01_Databox/01_Widgets/
Ce folder contient des exemples de widgets:

- écrites en typescript
- écrites en javascript
- un exemple de projet qui utilise React

## Points d'attention

- Pour bien comprendre comment écrire une bonne widget, il faut d'abord comprendre comment thingworx les charge dans le navigateur
  - Il récupère le code des widgets et les compile dans un fichier CombinedExtension.js, il faut faire attention à ne pas faire grossir ce fichier -> Il faut trouver des ressources sur internet qui explique ce mecanisme pour éviter les pièges (par exemple les chunks webpack semblent ne pas pouvoir marcher)
  - Bien comprendre le systeme de lazy loading des librairies externes pour minimiser la taille du CombinedExtensions.js dque Thingworx génère

- La valeur des propriétés peuvent être définies dans le composer (une valeur en dur) ou bindées depuis le résultat de l'execution d'un service
  - Lorsque les valeur sont statiques, la @property est initialisée avec la valeur définie dans le composer, on peut donc récupérer cette valeur dans afterRender
  - Lorsque qu'elle est bindée, le setter est appelé pour la propriété
  - La propriété peut avoir une valeur par défaut définie dans le composer ET être bindée
  - On ne controle pas dans quel ordre sont appelés les setters et le afterRender

# Instructions

**Ne pas imaginer le fonctionnement, poser des questions pour clarifier**
1 - Bien comprendre le cycle de vie des widgets
2 - Bien comprendre comment Thingworx utilise les widgets au runtime
3 - Bien comprendre comment sont écrites les widgets d'exemples
4 - Poser des questions si la compréhension n'est pas claire, s'il faut des précisions
5 - Proposer une skill performante qui produit du code de qualité (utilisation des références, templates, agents si nécessaire)
6 - Intégrer le mcp officiel chrome pour permettre à la skill de débugger une widget dans le navigateur - Pour débugger, il faudra indiquer à l'utilisateur de charger la widget dans thingworx, de naviguer vers un mashup qui contient la widget, et quand c'est fait la skill pourra débugger dans chrome

Faire des propositions pour créer une skill performante, ne pas hésiter à poser des questions, proposer des solutions.
