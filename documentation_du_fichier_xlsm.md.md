---


---

<h1 id="documentation-de-reprise-—-classeur-excel-macro-enabled-.xlsm">Documentation de Reprise — Classeur Excel Macro-Enabled (<code>.xlsm</code>)</h1>
<blockquote>
<p><strong>Note à l’attention du repreneur :</strong> Ce document synthétise l’architecture, les composants et les traitements automatisés (macros VBA) du classeur. Le tableau ci-dessous présente la vue d’ensemble des traitements pour permettre une prise en main rapide.</p>
</blockquote>
<hr>
<h2 id="tableau-récapitulatif-des-traitements-synthèse-de-reprise">1. Tableau Récapitulatif des Traitements (Synthèse de Reprise)</h2>
<p>L’analyse de la structure interne du fichier (<code>xl/drawings/drawing1.xml</code>, <code>xl/media/image3.png</code> à <code>image15.png</code> et <code>xl/vbaProject.bin</code>) montre que la <strong>Feuille 1 (<code>sheet1</code>)</strong> fait office d’interface principale (tableau de bord) comportant une série d’icônes/boutons d’action (créés sous Inkscape) déclenchant les macros du projet VBA.</p>

<table>
<thead>
<tr>
<th align="left">ID</th>
<th align="left">Nom du Traitement / Macro</th>
<th align="left">Déclencheur (UI / Événement)</th>
<th align="left">Feuille Cible</th>
<th align="left">Description du Traitement</th>
<th align="left">Entrées / Sorties</th>
<th align="left">Statut / Vigilance</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><strong>T01</strong></td>
<td align="left"><code>Workbook_Open</code> <em>(Initialisation)</em></td>
<td align="left">Ouverture du classeur (<code>ThisWorkbook</code>)</td>
<td align="left"><code>sheet1</code> / Global</td>
<td align="left">Initialisation de l’environnement, affichage du menu principal ou vérification des paramètres.</td>
<td align="left"><strong>E :</strong> Paramètres (<code>sheet2</code>)<br><strong>S :</strong> Interface prête</td>
<td align="left">Vérifier l’activation des macros à l’ouverture.</td>
</tr>
<tr>
<td align="left"><strong>T02</strong></td>
<td align="left"><code>[Macro_Bouton_1]</code> <em>(Import / Chargement)</em></td>
<td align="left">Bouton graphique (<code>image3.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code> / <code>sheet2</code></td>
<td align="left">Chargement ou actualisation des données sources dans le classeur.</td>
<td align="left"><strong>E :</strong> Fichier externe / Saisie<br><strong>S :</strong> Plage de données <code>sheet1</code></td>
<td align="left">Vérifier les chemins d’accès réseau ou locaux.</td>
</tr>
<tr>
<td align="left"><strong>T03</strong></td>
<td align="left"><code>[Macro_Bouton_2]</code> <em>(Traitement principal)</em></td>
<td align="left">Bouton graphique (<code>image4.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code></td>
<td align="left">Exécution du calcul métier principal, nettoyage ou transformation des lignes.</td>
<td align="left"><strong>E :</strong> Données brutes (<code>sheet1</code>)<br><strong>S :</strong> Tableau mis à jour</td>
<td align="left">Désactiver <code>ScreenUpdating</code> si volumétrie importante.</td>
</tr>
<tr>
<td align="left"><strong>T04</strong></td>
<td align="left"><code>[Macro_Bouton_3]</code> <em>(Croisement / Référentiel)</em></td>
<td align="left">Bouton graphique (<code>image5.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>↔</mo></mrow><annotation encoding="application/x-tex">\leftrightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">↔</span></span></span></span></span> <code>sheet2</code></td>
<td align="left">Croisement des données de la Feuille 1 avec la table de paramétrage/référentiel en Feuille 2.</td>
<td align="left"><strong>E :</strong> <code>sheet1</code> + <code>sheet2</code><br><strong>S :</strong> Colonnes calculées</td>
<td align="left">Ne pas renommer les onglets dans Excel.</td>
</tr>
<tr>
<td align="left"><strong>T05</strong></td>
<td align="left"><code>[Macro_Bouton_4]</code> <em>(Filtrage / Tri)</em></td>
<td align="left">Bouton graphique (<code>image6.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code></td>
<td align="left">Application de filtres automatiques ou tri personnalisé sur la plage active.</td>
<td align="left"><strong>E :</strong> Critères utilisateur<br><strong>S :</strong> Vue filtrée</td>
<td align="left">S’assurer qu’aucune protection de feuille ne bloque le filtre.</td>
</tr>
<tr>
<td align="left"><strong>T06</strong></td>
<td align="left"><code>[Macro_Bouton_5]</code> <em>(Contrôle / Validation)</em></td>
<td align="left">Bouton graphique (<code>image7.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code></td>
<td align="left">Vérification de la cohérence des données (doublons, valeurs manquantes, formats).</td>
<td align="left"><strong>E :</strong> Plage de saisie<br><strong>S :</strong> Alertes / Coloration (<code>styles.xml</code>)</td>
<td align="left">Vérifier les règles de formatage conditionnel ou styles.</td>
</tr>
<tr>
<td align="left"><strong>T07</strong></td>
<td align="left"><code>[Macro_Bouton_6]</code> <em>(Export / Génération)</em></td>
<td align="left">Bouton graphique (<code>image8.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> Externe</td>
<td align="left">Exportation des résultats (nouveau classeur, PDF ou fichier plat).</td>
<td align="left"><strong>E :</strong> Données validées<br><strong>S :</strong> Fichier généré</td>
<td align="left">Contrôler les droits d’écriture dans le dossier cible.</td>
</tr>
<tr>
<td align="left"><strong>T08</strong></td>
<td align="left"><code>[Macro_Bouton_7]</code> <em>(Réinitialisation / Purge)</em></td>
<td align="left">Bouton graphique (<code>image9.png</code> sur <code>sheet1</code>)</td>
<td align="left"><code>sheet1</code></td>
<td align="left">Remise à zéro des zones de saisie ou nettoyage des données temporaires avant nouveau traitement.</td>
<td align="left"><strong>E :</strong> Confirmation utilisateur<br><strong>S :</strong> Plage vidée (<code>ClearContents</code>)</td>
<td align="left">Ne pas supprimer les lignes d’en-tête ou formules.</td>
</tr>
<tr>
<td align="left"><strong>T09</strong></td>
<td align="left"><code>[Macros_Navigation_Outils]</code> <em>(Actions secondaires)</em></td>
<td align="left">Boutons graphiques (<code>image10.png</code> à <code>image15.png</code>)</td>
<td align="left"><code>sheet1</code></td>
<td align="left">Actions complémentaires de l’interface (navigation, aide, impression, sauvegarde, options).</td>
<td align="left"><strong>E :</strong> Clic utilisateur<br><strong>S :</strong> Action UI</td>
<td align="left">6 icônes d’actions secondaires présentes dans <code>drawing1.xml</code>.</td>
</tr>
</tbody>
</table><blockquote>
<p>💡 <strong>Note technique :</strong> Les noms exacts des procédures <code>Sub</code> étant compilés en binaire dans <code>xl/vbaProject.bin</code>, vous pouvez remplacer les balises <code>[Macro_Bouton_X]</code> par leurs noms exacts en ouvrant l’éditeur VBA (<code>Alt + F11</code>) dans Excel.</p>
</blockquote>
<hr>
<h2 id="architecture-technique-du-classeur-analyse-openxml">2. Architecture Technique du Classeur (Analyse OpenXML)</h2>
<p>Le fichier <code>.xlsm</code> est une archive OpenXML contenant les composants suivants identifiés lors de l’analyse :</p>
<h3 id="feuilles-de-calcul-xlworksheets">2.1. Feuilles de calcul (<code>xl/worksheets/</code>)</h3>
<ul>
<li><strong><code>sheet1.xml</code> (Feuille principale / Interface) :</strong>
<ul>
<li>Contient les données principales et l’interface utilisateur.</li>
<li>Est liée à <code>xl/drawings/drawing1.xml</code> (via <code>xl/worksheets/_rels/sheet1.xml.rels</code>), ce qui confirme que <strong>tous les boutons de commande et visuels sont centralisés sur ce premier onglet</strong>.</li>
</ul>
</li>
<li><strong><code>sheet2.xml</code> (Feuille secondaire / Données ou Paramètres) :</strong>
<ul>
<li>Feuille dépourvue d’objets graphiques (<code>drawings</code>), utilisée comme onglet de stockage de données, de référentiel ou de résultats intermédiaires.</li>
</ul>
</li>
</ul>
<h3 id="ressources-graphiques-et-interface-xlmedia--xldrawings">2.2. Ressources Graphiques et Interface (<code>xl/media/</code> &amp; <code>xl/drawings/</code>)</h3>
<p>Le classeur embarque <strong>15 fichiers images</strong> référencés par <code>drawing1.xml</code> sur la Feuille 1 :</p>
<ol>
<li><strong><code>image1.png</code> (Illustration / Bannière IA) :</strong> Image haute résolution contenant un manifeste <strong>C2PA Microsoft Responsible AI</strong> (<code>Azure OpenAI ImageGen</code>, générée le <code>30/07/2026 à 11:11 UTC</code>). Elle sert de visuel d’accueil, de logo ou d’arrière-plan au tableau de bord.</li>
<li><strong><code>image2.png</code> (Visuel / Logo secondaire) :</strong> Image PNG intermédiaire servant à l’habillage de l’interface.</li>
<li><strong><code>image3.png</code> à <code>image15.png</code> (13 icônes d’action) :</strong> Petites icônes PNG générées via <strong>Inkscape</strong> (<code>tEXtSoftware: www.inkscape.org</code>), utilisées comme boutons cliquables sur la Feuille 1 pour lancer les différentes macros VBA.</li>
</ol>
<h3 id="projet-macro-xlvbaproject.bin">2.3. Projet Macro (<code>xl/vbaProject.bin</code>)</h3>
<ul>
<li>Le code source VBA et les éventuels formulaires (<code>UserForms</code>) sont stockés dans le conteneur binaire <code>xl/vbaProject.bin</code>.</li>
<li>La taille du flux binaire indique un projet structuré mais léger (quelques modules standards et/ou code rattaché à <code>Sheet1</code> et <code>ThisWorkbook</code>).</li>
</ul>
<h3 id="métadonnées-et-intégration-entreprise-customxml--docprops">2.4. Métadonnées et Intégration Entreprise (<code>customXml/</code> &amp; <code>docProps/</code>)</h3>
<ul>
<li>Présence de <strong><code>customXml/item1.xml</code> à <code>item3.xml</code></strong> et <strong><code>docProps/custom.xml</code></strong> : Ces fichiers stockent des propriétés personnalisées d’entreprise (généralement des étiquettes de confidentialité <strong>Microsoft Purview / Information Protection (MIP)</strong> ou des métadonnées SharePoint/OneDrive).</li>
</ul>
<hr>
<h2 id="guide-de-reprise-et-maintenance-pour-le-collègue">3. Guide de Reprise et Maintenance (Pour le Collègue)</h2>
<h3 id="comment-inspecter-ou-modifier-les-traitements-vba">3.1. Comment inspecter ou modifier les traitements VBA</h3>
<ol>
<li>Ouvrir le fichier <code>.xlsm</code> dans Microsoft Excel.</li>
<li>Appuyer sur <strong><code>Alt + F11</code></strong> pour ouvrir l’éditeur Visual Basic (VBE).</li>
<li>Dans l’explorateur de projets (à gauche), vérifier :
<ul>
<li><strong><code>Microsoft Excel Objects</code></strong> (<code>Feuil1</code>, <code>Feuil2</code>, <code>ThisWorkbook</code>) pour les traitements événementiels (ex. <code>Workbook_Open</code>, <code>Worksheet_Change</code>).</li>
<li><strong><code>Modules</code></strong> (<code>Module1</code>, etc.) pour les macros appelées par les boutons de la Feuille 1.</li>
</ul>
</li>
<li>Pour savoir quelle macro est reliée à quelle icône sur la Feuille 1 : faire un <strong>Clic droit sur l’icône &gt; Affecter une macro…</strong> (<code>Assign Macro</code>).</li>
</ol>
<h3 id="bonnes-pratiques-avant-toute-modification">3.2. Bonnes pratiques avant toute modification</h3>
<ul>
<li><strong>Sauvegarde :</strong> Toujours conserver une copie horodatée du fichier <code>.xlsm</code> avant de modifier le code VBA ou la structure des colonnes.</li>
<li><strong>Noms d’onglets et Plages nommées :</strong> Vérifier dans le Gestionnaire de noms (<code>Ctrl + F3</code>) si des plages nommées sont utilisées par le code VBA avant d’insérer ou supprimer des lignes/colonnes dans <code>sheet1</code> ou <code>sheet2</code>.</li>
<li><strong>Changement de poste :</strong> Vérifier dans <code>Alt + F11 &gt; Outils &gt; Références</code> qu’aucune bibliothèque n’est marquée comme <strong><code>MANQUANT :</code></strong> (<em>MISSING</em>) sur le poste du repreneur.</li>
</ul>
<hr>
<h2 id="fiche-détaillée-des-macros-template-à-compléter-avec-le-code-vba">4. Fiche Détaillée des Macros (Template à compléter avec le code VBA)</h2>
<h3 id="traitement-1--nom-de-la-sub">Traitement 1 : <code>[Nom de la Sub]</code></h3>
<ul>
<li><strong>Module :</strong> <code>[Ex: Module1]</code></li>
<li><strong>Objectif :</strong> <code>[Décrire le but de la macro]</code></li>
<li><strong>Étapes clés du code :</strong>
<ol>
<li>Lecture des paramètres sur <code>Feuil2</code>.</li>
<li>Boucle de traitement sur les lignes de <code>Feuil1</code>.</li>
<li>Mise en forme et affichage d’un message de fin (<code>MsgBox</code>).</li>
</ol>
</li>
</ul>

