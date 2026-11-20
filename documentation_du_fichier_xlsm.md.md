---


---

<h1 id="documentation-technique-et-fonctionnelle-—-automatisation_scenarios_deterministes.xlsm">Documentation Technique et Fonctionnelle — <code>automatisation_scenarios_deterministes.xlsm</code></h1>
<blockquote>
<p><strong>Objectif de l’outil :</strong> Ce classeur Excel sert d’application (« Front-End » en plein écran) permettant d’automatiser la production périodique (<strong>Août</strong> et <strong>Décembre</strong>) du fichier des <strong>Scénarios Déterministes</strong>, la mise à jour des formules liées aux fichiers <strong>Consensus (Europe &amp; France)</strong>, ainsi que l’actualisation en chaîne des <strong>Maquettes Solveo (<code>1h</code> à <code>1s</code>)</strong> et des <strong>Fichiers d’Exports Solveo (<code>3_1h_export</code> à <code>3_1s_export</code>)</strong>.</p>
</blockquote>
<hr>
<h2 id="tableau-récapitulatif-de-tous-les-traitements-synthèse-de-reprise">1. Tableau Récapitulatif de Tous les Traitements (Synthèse de Reprise)</h2>
<p>Ce tableau recense l’intégralité des traitements (macros publiques, fonctions métier et procédures internes) présents dans le projet VBA, classés par phase fonctionnelle.</p>
<h3 id="orchestration-principale--paramétrage-de-la-période">1.1. Orchestration Principale &amp; Paramétrage de la Période</h3>

<table>
<thead>
<tr>
<th align="left">Phase</th>
<th align="left">Nom de la Procédure / Fonction</th>
<th align="left">Module</th>
<th align="left">Déclencheur</th>
<th align="left">Rôle et Logique du Traitement</th>
<th align="left">Entrées / Sorties / Plages impactées</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><strong>1. Init</strong></td>
<td align="left"><code>InitialiserPeriodeCible</code></td>
<td align="left"><code>modParametres</code></td>
<td align="left">Ouverture / Initialisation</td>
<td align="left">Compare la date du jour (<code>Date</code>) aux 3 échéances (<code>01/08/N</code>, <code>01/12/N</code>, <code>01/12/N-1</code>) et sélectionne automatiquement la période la plus proche. Déverrouille les feuilles et écrit l’année et le mois cible.</td>
<td align="left"><strong>Sorties :</strong> Plages nommées <code>ANNEE_CIBLE</code> (ex. <code>2026</code>) et <code>MOIS_CIBLE</code> (<code>"aout"</code> ou <code>"dec"</code>).</td>
</tr>
<tr>
<td align="left"><strong>1. Init</strong></td>
<td align="left"><code>LireAnneeCible</code> / <code>LireMoisCible</code></td>
<td align="left"><code>modParametres</code></td>
<td align="left">Appelé par <code>LancerTraitement</code></td>
<td align="left">Lit et valide la cohérence de la période cible saisie par l’utilisateur (année entière comprise entre <code>2023</code> et <code>2100</code>, mois égal à <code>"aout"</code> ou <code>"dec"</code>).</td>
<td align="left"><strong>Entrées :</strong> <code>ANNEE_CIBLE</code>, <code>MOIS_CIBLE</code>. Lève les erreurs <code>+1000</code> à <code>+1003</code> si invalide.</td>
</tr>
<tr>
<td align="left"><strong>2. Sélection</strong></td>
<td align="left"><code>SelectionnerFichierScenario</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left">Bouton UI <em>(Scénarios)</em></td>
<td align="left">Ouvre une boîte de dialogue fichier, vérifie que l’utilisateur n’a pas sélectionné l’outil lui-même, mémorise le chemin et affiche l’icône de validation verte.</td>
<td align="left"><strong>Sorties :</strong> Variable <code>CHEMIN_SCENARIO</code>, Nom caché <code>CHEMIN_SCENARIO_SAUVE</code>, Forme <code>sucess_scenarios</code>.</td>
</tr>
<tr>
<td align="left"><strong>2. Sélection</strong></td>
<td align="left"><code>SelectionnerFichiersConsensus</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left">Bouton UI <em>(Consensus)</em></td>
<td align="left">Permet la sélection multiple des fichiers Excel Consensus. Identifie automatiquement le fichier <strong>France</strong> (nom contenant <code>"france"</code>) et <strong>Europe</strong> (nom contenant <code>"euro"</code> ou <code>"zone"</code>).</td>
<td align="left"><strong>Sorties :</strong> <code>CHEMIN_CONSENSUS_EUROPE</code>, <code>CHEMIN_CONSENSUS_FRANCE</code>, Noms cachés <code>*_SAUVE</code>, Forme <code>sucess_concensus</code>.</td>
</tr>
<tr>
<td align="left"><strong>2. Sélection</strong></td>
<td align="left"><code>SelectionnerFichiersSolveo</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left">Bouton UI <em>(Maquettes Solveo)</em></td>
<td align="left">Sélection multiple des maquettes Solveo. Filtre les fichiers via <code>EstFichierSolveo1J</code> (seuls les fichiers préfixés par <code>1h</code> à <code>1s</code> suivis d’un espace ou <code>_</code> sont acceptés). Liste les fichiers rejetés.</td>
<td align="left"><strong>Sorties :</strong> Collection <code>FICHIERS_SOLVEO</code>, Noms cachés <code>CHEMIN_MAQUETTE_[H-S]_SAUVE</code>, Forme <code>sucess_maquettes</code>.</td>
</tr>
<tr>
<td align="left"><strong>2. Sélection</strong></td>
<td align="left"><code>SelectionnerFichiersExportSolveo</code></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left">Bouton UI <em>(Exports Solveo)</em></td>
<td align="left">Sélection multiple des fichiers d’export Solveo. Vérifie le format exact <code>3_1[h-s]_export.xls*</code> (11 caractères hors extension) et bloque les doublons via un <code>Scripting.Dictionary</code>.</td>
<td align="left"><strong>Sorties :</strong> Collection <code>FICHIERS_EXPORT_SOLVEO</code>, Noms cachés <code>CHEMIN_EXPORT_[H-S]_SAUVE</code>, Forme <code>sucess_maquettes</code>.</td>
</tr>
<tr>
<td align="left"><strong>2. Sélection</strong></td>
<td align="left"><code>SelectionnerRepertoireSortie</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left">Bouton UI <em>(Répertoire)</em></td>
<td align="left">Ouvre un sélecteur de dossier (<code>msoFileDialogFolderPicker</code>) pour définir le dossier racine où seront générés les livrables.</td>
<td align="left"><strong>Sorties :</strong> Variable <code>REPERTOIRE_SORTIE</code>, Nom caché <code>REPERTOIRE_SORTIE_SAUVE</code>, Forme <code>sucess_repertoire</code>.</td>
</tr>
<tr>
<td align="left"><strong>3. Exécution</strong></td>
<td align="left"><strong><code>LancerTraitement</code></strong></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><strong>Bouton Principal UI</strong></td>
<td align="left"><strong>Chef d’orchestre de bout en bout :</strong><br>1. Recharge et contrôle toutes les sélections.<br>2. Crée l’arborescence <code>&lt;Sortie&gt;\YYYYMMDD\</code> et <code>&lt;Sortie&gt;\YYYYMMDD\Consensus\</code>.<br>3. Copie les fichiers Scénarios et Consensus.<br>4. Exécute <code>TraiterPeriodeAout</code> ou <code>TraiterPeriodeDecembre</code>.<br>5. Lance <code>Application.CalculateFull</code> et sauvegarde le fichier Scénarios.<br>6. Enchaîne avec <code>TraiterFichiersSolveo</code> puis <code>TraiterFichiersExportSolveo</code>.<br>7. Restaure l’interface Excel, active le fichier généré et <strong>ferme l’outil</strong>.</td>
<td align="left"><strong>Sorties :</strong><br>- Dossier <code>YYYY0831</code> ou <code>YYYY1231</code><br>- <code>Scénarios déterministes_vYYYYMMDD.xls</code><br>- Maquettes <code>1h..1s</code> actualisées<br>- Exports <code>3_1h..3_1s_export</code> actualisés.</td>
</tr>
</tbody>
</table><hr>
<h3 id="traitements-métier-scénarios-consensus-maquettes-solveo-exports-solveo">1.2. Traitements Métier (Scénarios, Consensus, Maquettes Solveo, Exports Solveo)</h3>

<table>
<thead>
<tr>
<th align="left">Sous-Système</th>
<th align="left">Nom de la Procédure / Fonction</th>
<th align="left">Module</th>
<th align="left">Appelant</th>
<th align="left">Rôle et Logique du Traitement</th>
<th align="left">Plages / Fichiers Cibles</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><strong>Contrôles Période</strong></td>
<td align="left"><code>ControlerPeriodeAout</code></td>
<td align="left"><code>modControles</code></td>
<td align="left"><code>TraiterPeriodeAout</code></td>
<td align="left">Vérifie que la feuille source <code>(ANNEE_CIBLE - 1) &amp; "dec"</code> (ex. <code>2025dec</code>) existe dans le classeur Scénarios avant de créer <code>YYYYaout</code> (ex. <code>2026aout</code>). Si la feuille cible existe déjà, demande confirmation à l’utilisateur.</td>
<td align="left">Onglets <code>YYYY-1dec</code> et <code>YYYYaout</code> du classeur Scénarios.</td>
</tr>
<tr>
<td align="left"><strong>Consensus</strong></td>
<td align="left"><code>MettreAJourFormulesConsensus</code></td>
<td align="left"><code>modConsensus</code></td>
<td align="left">Traitement période (<code>Aout</code>/<code>Dec</code>)</td>
<td align="left">Parcourt la plage <strong><code>C73:M77</code></strong> de la feuille Scénario cible. Pour chaque formule contenant un <code>RECHERCHEV</code> vers un fichier <code>.xls</code> :<br>1. Identifie s’il s’agit du Consensus Europe ou France.<br>2. Trouve la colonne de la date la plus récente <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>≤</mo></mrow><annotation encoding="application/x-tex">\le</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.77194em; vertical-align: -0.13597em;"></span><span class="mrel">≤</span></span></span></span></span> date limite.<br>3. Réécrit le chemin du fichier, la colonne de fin de plage et l’indice de colonne du <code>RECHERCHEV</code>.</td>
<td align="left"><strong>Plage modifiée :</strong> <code>C73:M77</code> (Feuille Scénario).<br><strong>Plage lue :</strong> Lignes <code>1</code> à <code>15</code>, à partir de la colonne <code>B</code> (Fichiers Consensus).</td>
</tr>
<tr>
<td align="left"><strong>Consensus</strong></td>
<td align="left"><code>ObtenirDateLimiteConsensus</code></td>
<td align="left"><code>modConsensus</code></td>
<td align="left"><code>MettreAJourFormulesConsensus</code></td>
<td align="left">Définit la date plafond de recherche dans les fichiers Consensus :<br>- Si mois = <code>"aout"</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> <strong><code>31/07/ANNEE_CIBLE</code></strong><br>- Si mois = <code>"dec"</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> <strong><code>31/10/ANNEE_CIBLE</code></strong></td>
<td align="left">Retourne un type <code>Date</code>.</td>
</tr>
<tr>
<td align="left"><strong>Consensus</strong></td>
<td align="left"><code>TrouverColonneConsensus</code></td>
<td align="left"><code>modConsensus</code></td>
<td align="left"><code>MettreAJourFormulesConsensus</code></td>
<td align="left">Scanne l’en-tête de la feuille Consensus (lignes <code>1</code> à <code>15</code>, de la colonne <code>B</code> jusqu’à la dernière colonne) pour trouver la date maximale inférieure ou égale à <code>dateLimite</code>.</td>
<td align="left">Retourne le numéro de colonne (<code>Long</code>) et la ligne d’en-tête.</td>
</tr>
<tr>
<td align="left"><strong>Consensus</strong></td>
<td align="left"><code>RemplacerLienConsensus</code><br><code>RemplacerFinPlageRechercheV</code><br><code>RemplacerIndiceRechercheV</code></td>
<td align="left"><code>modConsensus</code></td>
<td align="left"><code>MettreAJourFormulesConsensus</code></td>
<td align="left">Utilise des expressions régulières (<code>VBScript.RegExp</code>) et du parsing de chaînes pour mettre à jour dynamiquement la formule <code>RECHERCHEV</code> locale sans casser sa structure.</td>
<td align="left">Formules de la plage <code>C73:M77</code>.</td>
</tr>
<tr>
<td align="left"><strong>Maquettes Solveo</strong></td>
<td align="left"><code>TraiterFichiersSolveo</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>LancerTraitement</code> (92% <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> 99%)</td>
<td align="left">Pour chaque maquette <code>1h</code> à <code>1s</code> :<br>1. Copie la maquette dans <code>&lt;Sortie&gt;\YYYYMMDD\</code>.<br>2. Ouvre la copie en lecture/écriture.<br>3. Remplace la liaison Excel vers le nouveau fichier <code>Scénarios déterministes_vYYYYMMDD.xls</code>.<br>4. Met à jour le nom d’onglet dans les formules (<code>B8:L22</code>).<br>5. Rafraîchit toutes les requêtes/connexions et recalcule.<br>6. Sauvegarde et ferme la maquette.</td>
<td align="left">Fichiers <code>1h*</code> à <code>1s*</code> copiés dans le dossier de la période.</td>
</tr>
<tr>
<td align="left"><strong>Maquettes Solveo</strong></td>
<td align="left"><code>RemplacerLiaisonScenario</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>TraiterFichiersSolveo</code></td>
<td align="left">Recherche dans <code>LinkSources(xlLinkTypeExcelLinks)</code> toute liaison contenant <code>"scénarios déterministes"</code> (avec ou sans accents) et la redirige via <code>ChangeLink</code> vers le nouveau fichier généré.</td>
<td align="left">Liaisons externes du classeur Solveo.</td>
</tr>
<tr>
<td align="left"><strong>Maquettes Solveo</strong></td>
<td align="left"><code>MettreAJourPeriodeMaquette</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>TraiterFichiersSolveo</code></td>
<td align="left">Parcourt <strong>toutes les feuilles</strong> de la maquette Solveo sur la plage <strong><code>B8:L22</code></strong>. Si une formule pointe vers <code>"Scénarios déterministes"</code>, remplace l’ancien nom d’onglet (pattern Regex <code>\](\d{4}(aout\|dec))'!</code>) par la nouvelle période (ex. <code>]2026aout'!</code>).</td>
<td align="left"><strong>Plage modifiée :</strong> <code>B8:L22</code> sur tous les onglets de chaque maquette <code>1h..1s</code>.</td>
</tr>
<tr>
<td align="left"><strong>Maquettes Solveo</strong></td>
<td align="left"><code>ActualiserClasseurSolveo</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>TraiterFichiersSolveo</code></td>
<td align="left">Exécute <code>classeurSolveo.RefreshAll</code> et boucle (avec <code>CalculateUntilAsyncQueriesDone</code> et <code>DoEvents</code>) tant qu’une connexion <code>OLEDB</code>, <code>ODBC</code>, <code>QueryTable</code> ou <code>ListObject</code> est en cours de rafraîchissement. <strong>Timeout de sécurité fixé à 300 secondes (5 min).</strong></td>
<td align="left">Connexions de données externes des maquettes Solveo.</td>
</tr>
<tr>
<td align="left"><strong>Exports Solveo</strong></td>
<td align="left"><code>TraiterFichiersExportSolveo</code></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>LancerTraitement</code> (97% <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> 99%)</td>
<td align="left">Pour chaque fichier <code>3_1[h-s]_export</code> :<br>1. Vérifie que la maquette correspondante <code>1[h-s]</code> a bien été générée dans le dossier de sortie.<br>2. Copie le fichier d’export dans le dossier de sortie.<br>3. Appelle <code>MettreAJourExportSolveo</code>, sauvegarde et ferme.</td>
<td align="left">Fichiers <code>3_1h_export*</code> à <code>3_1s_export*</code>.</td>
</tr>
<tr>
<td align="left"><strong>Exports Solveo</strong></td>
<td align="left"><code>MettreAJourExportSolveo</code></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>TraiterFichiersExportSolveo</code></td>
<td align="left">1. Contrôle la présence obligatoire des 4 onglets : <strong><code>Inflation___taux</code></strong>, <strong><code>Indices_actions</code></strong>, <strong><code>Immeubles</code></strong>, <strong><code>Indices_change</code></strong>.<br>2. Appelle <code>RemplacerLiaisonMaquetteExport</code> pour pointer vers la nouvelle maquette <code>1[h-s]</code> du dossier de sortie.<br>3. Appelle <code>MettreAJourDatesExport</code> sur les 4 onglets.</td>
<td align="left">Les 4 onglets métiers du classeur d’export Solveo.</td>
</tr>
<tr>
<td align="left"><strong>Exports Solveo</strong></td>
<td align="left"><code>MettreAJourDatesExport</code></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>MettreAJourExportSolveo</code></td>
<td align="left">Détermine la dernière ligne non vide de la feuille. À partir de la <strong>ligne 6 (<code>PREMIERE_LIGNE_DATES = 6</code>)</strong>, génère en <strong>colonne A</strong> une série de dates mensuelles commençant au <strong><code>01/12/(ANNEE_CIBLE - 1)</code></strong> (incrément d’1 mois par ligne) au format <code>"mm/yyyy"</code>.</td>
<td align="left"><strong>Plage modifiée :</strong> <code>A6:A&lt;dernière_ligne&gt;</code> sur les 4 feuilles d’export.</td>
</tr>
<tr>
<td align="left"><strong>Exports Solveo</strong></td>
<td align="left"><code>CalculerFormulesExport</code></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left">Utilitaire interne</td>
<td align="left">Recalcule uniquement les cellules contenant des formules (<code>SpecialCells(xlCellTypeFormulas)</code>) à partir de la <strong>ligne 19 (<code>PREMIERE_LIGNE_FORMULES = 19</code>)</strong> jusqu’à la dernière ligne/colonne.</td>
<td align="left">Plage <code>A19:&lt;DernièreCol&gt;&lt;DernièreLigne&gt;</code>.</td>
</tr>
</tbody>
</table><hr>
<h3 id="traitements-liés-à-linterface-graphique-ui-persistance--formulaires">1.3. Traitements liés à l’Interface Graphique (UI), Persistance &amp; Formulaires</h3>

<table>
<thead>
<tr>
<th align="left">Catégorie</th>
<th align="left">Nom de la Procédure / Fonction</th>
<th align="left">Module / Form</th>
<th align="left">Rôle et Fonctionnement</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><strong>Mode Applicatif</strong></td>
<td align="left"><code>Set_interface</code></td>
<td align="left"><code>Interface</code></td>
<td align="left">Transforme Excel en application plein écran : masque la barre de formule, la barre d’état, le Ruban (<code>SHOW.TOOLBAR("Ribbon",False)</code>), les en-têtes de lignes/colonnes, les onglets et les barres de défilement. Masque la feuille secondaire en <code>xlSheetVeryHidden</code> et protège la structure du classeur.</td>
</tr>
<tr>
<td align="left"><strong>Mode Applicatif</strong></td>
<td align="left"><code>ResetInterface</code></td>
<td align="left"><code>Interface</code></td>
<td align="left"><strong>Procédure de secours / sortie :</strong> Réaffiche le Ruban Excel, la barre de formule, les en-têtes, les onglets, les barres de défilement, rend visible la feuille masquée (<code>NOM_FEUILLE_A_MASQUER</code>) et réactive les événements.</td>
</tr>
<tr>
<td align="left"><strong>Navigation</strong></td>
<td align="left"><code>InitialiserBoutons</code><br><code>CliquerBoutonNavigation</code><br><code>ActiverBouton</code></td>
<td align="left"><code>Interface</code></td>
<td align="left">Gère le menu de navigation à 3 onglets visuels (<code>home</code>, <code>history</code>, <code>settings</code>). Permute dynamiquement la visibilité et l’ordre d’empilement (<code>ZOrder</code>) des formes <code>*_icon_active</code>, <code>*_icon_disabled</code>, <code>*_active_background</code> et applique les couleurs de texte (<code>4690964</code> actif / <code>6317149</code> inactif).</td>
</tr>
<tr>
<td align="left"><strong>Sécurité</strong></td>
<td align="left"><code>ProtectAllSheets</code><br><code>UnProtectAllSheets</code></td>
<td align="left"><code>Interface</code></td>
<td align="left">Verrouille / déverrouille toutes les feuilles avec <code>UserInterfaceOnly:=True</code> (permet au code VBA de modifier les cellules sans déprotéger, tout en bloquant l’utilisateur) ainsi que la structure du classeur via la constante <code>PASSWORD</code>.</td>
</tr>
<tr>
<td align="left"><strong>Indicateurs UI</strong></td>
<td align="left"><code>AfficherObjet</code> / <code>MasquerObjet</code></td>
<td align="left"><code>Interface</code></td>
<td align="left">Affiche ou masque les pastilles de validation (<code>sucess_repertoire</code>, <code>sucess_concensus</code>, <code>sucess_scenarios</code>, <code>sucess_maquettes</code>) sur la feuille principale. <em>(Note : la macro <code>qdlmkfsj</code> est une procédure de test rapide de ces affichages).</em></td>
</tr>
<tr>
<td align="left"><strong>Persistance</strong></td>
<td align="left"><code>ChargerSelectionsSauvegardees</code><br><code>EnregistrerValeurNom</code><br><code>EnregistrerSelectionsSolveo</code></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left">Sauvegarde les chemins sélectionnés par l’utilisateur directement dans des <strong>Noms définis masqués (<code>Visible:=False</code>)</strong> à l’intérieur du classeur (<code>CHEMIN_SCENARIO_SAUVE</code>, <code>CHEMIN_MAQUETTE_H_SAUVE</code>, etc.) pour les retrouver d’une session à l’autre.</td>
</tr>
<tr>
<td align="left"><strong>Barre de Progression</strong></td>
<td align="left"><code>MettreAJour</code></td>
<td align="left"><code>frmProgression</code></td>
<td align="left">Formulaire non-modal (<code>vbModeless</code>) affichant l’avancement en pourcentage (<code>0 %</code> à <code>100 %</code>), une barre visuelle (<code>lblProgression.Width</code> sur une base max de <code>462</code> pt) et le libellé de l’étape en cours. Empêche la fermeture manuelle via la croix rouge (<code>UserForm_QueryClose</code>).</td>
</tr>
<tr>
<td align="left"><strong>Confirmation</strong></td>
<td align="left"><code>UserForm_Initialize</code></td>
<td align="left"><code>frmEcrasement</code></td>
<td align="left">Boîte de dialogue personnalisée aux couleurs de la charte graphique (<code>RGB(20, 75, 62)</code>) destinée à confirmer le remplacement de fichiers existants.</td>
</tr>
<tr>
<td align="left"><strong>Aide</strong></td>
<td align="left"><code>OuvrirGuideUtilisation</code></td>
<td align="left"><code>Interface</code></td>
<td align="left">Affiche le formulaire modal <code>Usage_Userform</code> contenant le guide utilisateur.</td>
</tr>
</tbody>
</table><hr>
<h2 id="cinématique-détaillée-du-traitement-principal-lancertraitement">2. Cinématique détaillée du Traitement Principal (<code>LancerTraitement</code>)</h2>
<p>Lorsqu’un utilisateur clique sur le bouton de lancement, la procédure <code>LancerTraitement</code> (dans <code>modSelectionFichier</code>) exécute séquentiellement les opérations suivantes :</p>
<ol>
<li>
<p><strong>Chargement et Validation des Prérequis (0 %) :</strong></p>
<ul>
<li>Récupération des chemins enregistrés dans les noms masqués du classeur (<code>ChargerSelectionsSauvegardees</code> et <code>ChargerSelectionsSolveoSauvegardees</code>).</li>
<li>Vérification de l’existence physique sur le disque de tous les fichiers : Scénarios, Consensus Europe, Consensus France, dossier de sortie, au moins une maquette Solveo (<code>1h..1s</code>) et au moins un export Solveo (<code>3_1h..3_1s_export</code>).</li>
<li>Lecture de <code>ANNEE_CIBLE</code> et <code>MOIS_CIBLE</code>.</li>
</ul>
</li>
<li>
<p><strong>Création de l’Arborescence de Sortie :</strong></p>
<ul>
<li>Si <code>MOIS_CIBLE = "aout"</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> création du sous-dossier <code>&lt;REPERTOIRE_SORTIE&gt;\YYYY0831\</code>.</li>
<li>Si <code>MOIS_CIBLE = "dec"</code> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> création du sous-dossier <code>&lt;REPERTOIRE_SORTIE&gt;\YYYY1231\</code>.</li>
<li>Création du sous-dossier <code>&lt;REPERTOIRE_SORTIE&gt;\YYYYMMDD\Consensus\</code>.</li>
<li>Définition du nom du fichier cible : <code>Scénarios déterministes_vYYYYMMDD.xls</code>.</li>
<li>Demande de confirmation d’écrasement si les fichiers existent déjà dans la destination.</li>
</ul>
</li>
<li>
<p><strong>Verrouillage de l’Environnement Excel &amp; Copie (5 % à 65 %) :</strong></p>
<ul>
<li>Passage d’Excel en mode haute performance : <code>Calculation = xlCalculationManual</code>, <code>ScreenUpdating = False</code>, <code>EnableEvents = False</code>, <code>DisplayAlerts = False</code>.</li>
<li>Copie des 3 fichiers sources vers l’arborescence cible et ouverture en mémoire (<code>classeurScenario</code> en écriture, <code>classeurConsensusEurope</code> et <code>classeurConsensusFrance</code> en lecture seule).</li>
</ul>
</li>
<li>
<p><strong>Traitement Métier du Classeur Scénarios &amp; Consensus (75 % à 92 %) :</strong></p>
<ul>
<li>Appel de <code>TraiterPeriodeAout</code> ou <code>TraiterPeriodeDecembre</code> :
<ul>
<li>Contrôle de l’onglet de départ (ex. vérification de la présence de <code>2025dec</code> via <code>ControlerPeriodeAout</code>).</li>
<li>Création/mise à jour de l’onglet cible (<code>YYYYaout</code> ou <code>YYYYdec</code>).</li>
<li>Appel de <code>MettreAJourFormulesConsensus</code> : mise à jour automatique des <code>RECHERCHEV</code> de la plage <strong><code>C73:M77</code></strong> vers les nouveaux fichiers Consensus copiés dans <code>\Consensus\</code>, en sélectionnant la colonne de date la plus proche <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>≤</mo></mrow><annotation encoding="application/x-tex">\le</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.77194em; vertical-align: -0.13597em;"></span><span class="mrel">≤</span></span></span></span></span> <code>31/07/YYYY</code> (pour août) ou <code>31/10/YYYY</code> (pour décembre).</li>
</ul>
</li>
<li>Recalcul complet de tous les classeurs ouverts (<code>Application.CalculateFull</code>) et sauvegarde du fichier <code>Scénarios déterministes_vYYYYMMDD.xls</code>.</li>
</ul>
</li>
<li>
<p><strong>Actualisation en Chaîne des Maquettes Solveo <code>1h</code> à <code>1s</code> (92 % à 97 %) :</strong></p>
<ul>
<li>Via <code>TraiterFichiersSolveo</code> :
<ul>
<li>Copie de chaque maquette dans <code>&lt;REPERTOIRE_SORTIE&gt;\YYYYMMDD\</code>.</li>
<li>Redirection de la liaison externe vers <code>Scénarios déterministes_vYYYYMMDD.xls</code>.</li>
<li>Remplacement par Regex du nom de l’onglet cible dans toutes les formules de la plage <strong><code>B8:L22</code></strong> de chaque feuille.</li>
<li>Exécution de <code>RefreshAll</code> (attente de fin des requêtes asynchrones OLEDB/ODBC/QueryTables avec timeout de 5 minutes) puis recalcul feuille par feuille et sauvegarde.</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Actualisation en Chaîne des Exports Solveo <code>3_1h_export</code> à <code>3_1s_export</code> (97 % à 99 %) :</strong></p>
<ul>
<li>Via <code>TraiterFichiersExportSolveo</code> :
<ul>
<li>Association de chaque fichier <code>3_1[x]_export</code> avec la maquette <code>1[x]</code> fraîchement générée dans le dossier de sortie.</li>
<li>Copie et ouverture du fichier d’export.</li>
<li>Vérification de la présence des 4 feuilles : <code>Inflation___taux</code>, <code>Indices_actions</code>, <code>Immeubles</code>, <code>Indices_change</code>.</li>
<li>Changement de la liaison Excel (<code>ChangeLink</code>) vers la nouvelle maquette <code>1[x]</code> du répertoire de sortie.</li>
<li>Régénération du calendrier mensuel en colonne <code>A</code> (à partir de <code>A6</code>) en partant de <code>12/(ANNEE_CIBLE - 1)</code>.</li>
<li>Sauvegarde et fermeture du classeur d’export.</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Clôture et Sortie (100 %) :</strong></p>
<ul>
<li>Fermeture des fichiers Consensus.</li>
<li>Masquage des icônes de succès (<code>sucess_*</code>), restauration de l’affichage standard d’Excel (<code>ResetInterface</code>), affichage au premier plan du classeur <code>Scénarios déterministes_vYYYYMMDD.xls</code> ouvert sur l’onglet de la nouvelle période, et <strong>fermeture automatique sans sauvegarde du classeur outil <code>.xlsm</code></strong>.</li>
</ul>
</li>
</ol>
<hr>
<h2 id="règles-de-gestion-et-conventions-de-nommage-strictes">3. Règles de Gestion et Conventions de Nommage Strictes</h2>
<p>Si un futur utilisateur rencontre un rejet lors de la sélection des fichiers, voici les règles exactes codées en dur dans le VBA :</p>
<ol>
<li>
<p><strong>Fichiers Consensus (<code>SelectionnerFichiersConsensus</code>) :</strong></p>
<ul>
<li>L’utilisateur doit sélectionner les 2 fichiers simultanément.</li>
<li>Le fichier France <strong>doit</strong> contenir la chaîne <code>"france"</code> (insensible à la casse) dans son nom.</li>
<li>Le fichier Europe <strong>doit</strong> contenir la chaîne <code>"euro"</code> ou <code>"zone"</code> dans son nom.</li>
</ul>
</li>
<li>
<p><strong>Maquettes Solveo (<code>EstFichierSolveo1J</code> / <code>ExtraireCodeMaquetteSolveo</code>) :</strong></p>
<ul>
<li>Le nom du fichier (après suppression des espaces de début/fin) doit faire au moins 3 caractères.</li>
<li>Caractère 1 : <strong><code>1</code></strong></li>
<li>Caractère 2 : Une lettre comprise entre <strong><code>h</code></strong> et <strong><code>s</code></strong> inclus (<code>1h</code>, <code>1i</code>, <code>1j</code>, …, <code>1s</code>).</li>
<li>Caractère 3 : Un espace (<code>" "</code>) ou un underscore (<code>"_"</code>).</li>
<li><em>Exemples valides :</em> <code>1h_maquette.xlsx</code>, <code>1s modele.xlsm</code>.</li>
</ul>
</li>
<li>
<p><strong>Exports Solveo (<code>ExtraireCodeExportSolveo</code>) :</strong></p>
<ul>
<li>Le nom du fichier <strong>sans son extension</strong> doit mesurer <strong>exactement 11 caractères</strong>.</li>
<li>Il doit commencer par <strong><code>3_1</code></strong>, suivi d’une lettre entre <strong><code>h</code></strong> et <strong><code>s</code></strong>, suivi de <strong><code>_export</code></strong>.</li>
<li><em>Exemples valides :</em> <code>3_1h_export.xlsx</code>, <code>3_1s_export.xls</code>.</li>
<li>Chaque export <code>3_1[x]_export</code> exige que la maquette <code>1[x]</code> correspondante ait été sélectionnée et traitée.</li>
</ul>
</li>
</ol>
<hr>
<h2 id="référentiel-des-codes-derreurs-personnalisés-err.raise">4. Référentiel des Codes d’Erreurs Personnalisés (<code>Err.Raise</code>)</h2>
<p>Le code utilise des numéros d’erreurs personnalisés basés sur <code>vbObjectError</code> pour faciliter le diagnostic :</p>

<table>
<thead>
<tr>
<th align="left">Code (<code>vbObjectError + X</code>)</th>
<th align="left">Module</th>
<th align="left">Procédure</th>
<th align="left">Cause de l’erreur</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><strong><code>+ 1000</code> à <code>+ 1002</code></strong></td>
<td align="left"><code>modParametres</code></td>
<td align="left"><code>LireAnneeCible</code></td>
<td align="left"><code>ANNEE_CIBLE</code> non numérique, non entière, ou hors de la plage <code>[2023 ; 2100]</code>.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 1003</code></strong></td>
<td align="left"><code>modParametres</code></td>
<td align="left"><code>LireMoisCible</code></td>
<td align="left"><code>MOIS_CIBLE</code> différent de <code>"aout"</code> et <code>"dec"</code>.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 2000</code></strong></td>
<td align="left"><code>modConsensus</code></td>
<td align="left"><code>ObtenirDateLimiteConsensus</code></td>
<td align="left">Mois cible Consensus invalide.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 2001</code> à <code>+ 2003</code></strong></td>
<td align="left"><code>modConsensus</code></td>
<td align="left"><code>Remplacer*</code></td>
<td align="left">Structure de la formule <code>RECHERCHEV</code> dans <code>C73:M77</code> non reconnue par les Regex (lien externe, indice de colonne ou plage <code>$B$x:$Z$y</code> introuvable).</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3000</code> à <code>+ 3003</code></strong></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>CopierFichier</code> / <code>LancerTraitement</code></td>
<td align="left">Fichier source identique à la destination, ou échec d’ouverture des copies Scénarios / Consensus.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3100</code> à <code>+ 3105</code></strong></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>TraiterFichiersSolveo</code></td>
<td align="left">Maquette source introuvable, échec de copie, ouverture en lecture seule, ou échec de sauvegarde.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3110</code></strong></td>
<td align="left"><code>modSelectionFichier</code></td>
<td align="left"><code>ActualiserClasseurSolveo</code></td>
<td align="left">Dépassement du délai maximal (Timeout &gt; 300 secondes) lors du <code>RefreshAll</code> d’une maquette Solveo.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3200</code> à <code>+ 3204</code></strong></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>TraiterFichiersExportSolveo</code></td>
<td align="left">Nom d’export invalide, source introuvable, lecture seule ou échec d’enregistrement de l’export.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3210</code> à <code>+ 3213</code></strong></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>MettreAJourExportSolveo</code></td>
<td align="left">Absence de l’une des 4 feuilles obligatoires (<code>Inflation___taux</code>, <code>Indices_actions</code>, <code>Immeubles</code>, <code>Indices_change</code>).</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3230</code></strong></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>CopierFichierExport</code></td>
<td align="left">Fichier d’export source et destination identiques.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 3240</code> à <code>+ 3242</code></strong></td>
<td align="left"><code>GestionExportsSolveo</code></td>
<td align="left"><code>RemplacerLiaisonMaquetteExport</code></td>
<td align="left">Nom de maquette invalide, aucune liaison Excel dans l’export, ou aucune liaison vers une ancienne maquette <code>1[x]</code> trouvée.</td>
</tr>
<tr>
<td align="left"><strong><code>+ 4000</code> à <code>+ 4015</code></strong></td>
<td align="left"><code>Interface</code></td>
<td align="left"><code>ActiverBouton</code> / <code>AppliquerCouleurTexte</code></td>
<td align="left">Forme graphique (<code>Shape</code>) d’un bouton de navigation introuvable dans <code>Feuil1</code> ou impossible à colorer.</td>
</tr>
</tbody>
</table><hr>
<h2 id="points-dattention-pour-la-maintenance-notes-au-repreneur">5. Points d’Attention pour la Maintenance (Notes au Repreneur)</h2>
<ol>
<li>
<p><strong>Plages de cellules codées en dur (« Hardcodées ») :</strong></p>
<ul>
<li><strong><code>C73:M77</code></strong> (<code>modConsensus.MettreAJourFormulesConsensus</code>) : Si la structure de la feuille des Scénarios Déterministes évolue et que le tableau des formules Consensus est déplacé (ajout/suppression de lignes au-dessus de la ligne 73), il faudra impérativement adapter cette plage dans le code.</li>
<li><strong><code>B8:L22</code></strong> (<code>modSelectionFichier.MettreAJourPeriodeMaquette</code>) : Seules les formules situées dans ce rectangle sur chaque feuille des maquettes Solveo voient leur référence d’onglet (<code>YYYYaout</code> / <code>YYYYdec</code>) mise à jour.</li>
<li><strong>Ligne <code>6</code> (<code>PREMIERE_LIGNE_DATES</code>) et Ligne <code>19</code> (<code>PREMIERE_LIGNE_FORMULES</code>)</strong> dans <code>GestionExportsSolveo</code> : Définissent le point de départ des dates en colonne A et des formules dans les fichiers <code>3_1*_export</code>.</li>
<li><strong>Lignes <code>1</code> à <code>15</code></strong> (<code>modConsensus</code>) : Zone de recherche des dates d’en-tête dans les fichiers Consensus Europe et France.</li>
</ul>
</li>
<li>
<p><strong>Boîte de dialogue de débogage laissée active :</strong></p>
<ul>
<li>Dans <code>modSelectionFichier.LancerTraitement</code> (au tout début), la ligne suivante est active :<pre class=" language-vba"><code class="prism  language-vba">MsgBox "REPERTOIRE_SORTIE = " &amp; REPERTOIRE_SORTIE, vbInformation, "Valeur actuelle"
</code></pre>
</li>
<li>Il s’agit manifestement d’un <code>MsgBox</code> de débogage qui peut être commenté ou supprimé pour fluidifier l’expérience utilisateur.</li>
</ul>
</li>
<li>
<p><strong>Fermeture automatique de l’application en fin de traitement :</strong></p>
<ul>
<li>À la fin de <code>LancerTraitement</code>, l’instruction <code>ThisWorkbook.Close SaveChanges:=False</code> ferme automatiquement l’outil <code>.xlsm</code> pour ne laisser ouvert que le fichier <code>Scénarios déterministes_vYYYYMMDD.xls</code> généré.</li>
<li>Si vous modifiez le code VBA dans l’éditeur (<code>Alt + F11</code>), <strong>pensez toujours à sauvegarder le classeur <code>.xlsm</code> avant de tester <code>LancerTraitement</code></strong>, sinon vos modifications de code non enregistrées seront perdues lors de la fermeture automatique !</li>
</ul>
</li>
<li>
<p><strong>Déblocage de l’interface Excel en cas d’interruption :</strong></p>
<ul>
<li>Si l’exécution est interrompue manuellement en mode débogage alors que le ruban et les onglets sont masqués, exécutez simplement la macro <strong><code>ResetInterface</code></strong> et <strong><code>UnProtectAllSheets</code></strong> (dans le module <code>Interface</code>) depuis la fenêtre Exécution (<code>Ctrl + G</code>) de l’éditeur VBA pour retrouver l’interface complète d’Excel.</li>
</ul>
</li>
</ol>

