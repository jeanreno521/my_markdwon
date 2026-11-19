---


---

<h1 id="documentation-détaillée-des-macros-vba">Documentation détaillée des macros VBA</h1>
<h2 id="classeur-analysé">Classeur analysé</h2>
<ul>
<li><strong>Fichier :</strong> <code>copier_valeur.xlsm</code></li>
<li><strong>Feuille de paramétrage :</strong> <code>param</code></li>
<li><strong>Objet général :</strong> alimenter plusieurs classeurs de scénarios à partir d’un classeur source, puis rafraîchir les fichiers sources et mettre à jour les liaisons des fichiers dépendants.</li>
<li><strong>Principe de localisation :</strong> tous les fichiers sont recherchés dans le <strong>même dossier que le classeur lanceur</strong> (<code>ThisWorkbook.Path</code>).</li>
</ul>
<h2 id="structure-du-classeur">1. Structure du classeur</h2>
<p>Le projet VBA contient les composants suivants :</p>

<table>
<thead>
<tr>
<th>Composant</th>
<th>Type</th>
<th>Rôle</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ThisWorkbook</code></td>
<td>Module de classe</td>
<td>Aucun code événementiel.</td>
</tr>
<tr>
<td><code>Feuil1</code></td>
<td>Module de feuille</td>
<td>Aucun code événementiel.</td>
</tr>
<tr>
<td><code>Module_traitement_31h_31s</code></td>
<td>Module standard</td>
<td>Copie des onglets du fichier export source vers les fichiers <code>3_1h</code> à <code>3_1s</code>.</td>
</tr>
<tr>
<td><code>Module_Traitement_1H_1R</code></td>
<td>Module standard</td>
<td>Copie de plages précises vers les maquettes <code>1h</code> à <code>1s</code>, avec transformations complémentaires.</td>
</tr>
<tr>
<td><code>Module1</code></td>
<td>Module standard</td>
<td>Variante de débogage du traitement <code>1h-1s</code>; contient des fonctions dupliquées.</td>
</tr>
<tr>
<td><code>Module2</code></td>
<td>Module standard</td>
<td>Rafraîchissement des sources et mise à jour des liaisons des fichiers dépendants.</td>
</tr>
</tbody>
</table><h2 id="paramètres-attendus-dans-la-feuille-param">2. Paramètres attendus dans la feuille <code>param</code></h2>

<table>
<thead>
<tr>
<th>Zone</th>
<th>Contenu</th>
<th>Utilisée par</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>B2</code></td>
<td>Nom du fichier export source</td>
<td>Macros de copie <code>3_1h-3_1s</code> et <code>1h-1s</code></td>
</tr>
<tr>
<td><code>A2:A...</code></td>
<td>Liste des fichiers cibles <code>3_1h_export.xlsx</code> à <code>3_1s_export.xlsx</code></td>
<td><code>CopierOngletsVersFichiersCibles_31h_31s</code></td>
</tr>
<tr>
<td><code>C2:C...</code></td>
<td>Liste des maquettes cibles <code>1h</code> à <code>1s</code></td>
<td><code>CopierDonneesVersFichiersCibles_1h_1s</code>, <code>debuggeur_1h_1s</code>, <code>MettreAJourToutesLesLiaisons</code></td>
</tr>
<tr>
<td><code>D2</code></td>
<td>Valeur de l’année <code>n</code> (ex. <code>2026</code>)</td>
<td>Modification de cellules de contrôle dans les maquettes cibles</td>
</tr>
<tr>
<td><code>E2</code></td>
<td>Indicateur de mise à jour des liaisons</td>
<td>Présent dans la feuille; le code principal de <code>Module2</code> pilote directement les mises à jour via les colonnes A et C</td>
</tr>
</tbody>
</table><h2 id="macro-copierongletsversfichierscibles_31h_31s">3. Macro <code>CopierOngletsVersFichiersCibles_31h_31s</code></h2>
<h3 id="finalité">Finalité</h3>
<p>Ouvrir une fois le fichier source indiqué en <code>B2</code>, parcourir les fichiers cibles de la colonne A, remplacer leurs données par celles du fichier source, puis sauvegarder chaque cible.</p>
<h3 id="traitements-réalisés">Traitements réalisés</h3>
<ol>
<li>Vérifie que <code>param!B2</code> est renseignée.</li>
<li>Détermine la dernière ligne non vide de la colonne A.</li>
<li>Désactive temporairement le rafraîchissement écran, les alertes, les événements et le calcul automatique.</li>
<li>Ouvre le fichier source sans mettre à jour ses liens.</li>
<li>Pour chaque cible de <code>A2:A...</code> :
<ul>
<li>ouvre le fichier sans mise à jour des liens;</li>
<li>copie intégralement les onglets <code>Courbes_taux</code>, <code>Spreads_courbes_taux</code>, <code>Spreads_de_change</code> et <code>Indices_change</code>;</li>
<li>traite partiellement <code>Inflation___taux</code>, <code>Indices_actions</code> et <code>Immeubles</code>;</li>
<li>sauvegarde et ferme le fichier cible.</li>
</ul>
</li>
<li>Ferme le fichier source sans modification.</li>
<li>Restaure les paramètres Excel et affiche un rapport avec le nombre de succès, d’erreurs et la durée.</li>
</ol>
<h3 id="fonctions-appelées">Fonctions appelées</h3>
<ul>
<li><code>CopierOngletComplet</code> : efface la feuille cible, puis copie le <code>UsedRange</code> source à partir de <code>A1</code>.</li>
<li><code>CopierPlageEnteteClassique</code> : remplace la colonne A et les en-têtes des lignes 1 à 18 à partir de la colonne B.</li>
<li><code>CopierPlageEnteteInflation</code> : traitement spécialisé de <code>Inflation___taux</code>; nettoie les anciennes zones puis recopie la colonne A, les en-têtes et la matrice de données à partir de <code>F19</code>.</li>
</ul>
<h3 id="effets-et-risques">Effets et risques</h3>
<ul>
<li>Les contenus existants des feuilles cibles sont supprimés ou remplacés.</li>
<li>Les fichiers sont sauvegardés automatiquement, sans demande de confirmation.</li>
<li>Les noms d’onglets doivent être identiques dans la source et les cibles.</li>
<li>Une cible absente, verrouillée ou incompatible est comptée en erreur.</li>
</ul>
<h2 id="macro-copierdonneesversfichierscibles_1h_1s">4. Macro <code>CopierDonneesVersFichiersCibles_1h_1s</code></h2>
<h3 id="finalité-1">Finalité</h3>
<p>Transférer des plages déterminées du fichier source vers chaque maquette listée en colonne C, puis appliquer des ajustements complémentaires dans les feuilles cibles.</p>
<h3 id="déroulement">Déroulement</h3>
<ol>
<li>Lit le fichier source en <code>B2</code>.</li>
<li>Parcourt les fichiers cibles de <code>C2:C...</code>.</li>
<li>Exécute une série de copies de plages avec contrôle des dimensions.</li>
<li>Traite la feuille <code>Spread</code> à partir de <code>Inflation___taux</code>.</li>
<li>Applique des modifications ciblées dans les maquettes.</li>
<li>Sauvegarde chaque cible et affiche un rapport d’exécution.</li>
</ol>
<h3 id="mappages-explicitement-définis">Mappages explicitement définis</h3>

<table>
<thead>
<tr>
<th align="right">Étape</th>
<th>Source</th>
<th>Destination</th>
<th>Particularité</th>
</tr>
</thead>
<tbody>
<tr>
<td align="right">1</td>
<td><code>Inflation___taux!A4:A630</code></td>
<td><code>Taux!I9:I635</code></td>
<td>Copie avec traitement/format prévu par l’argument booléen <code>True</code></td>
</tr>
<tr>
<td align="right">2</td>
<td><code>Inflation___taux!D6:E18</code></td>
<td><code>Taux!J11:K23</code></td>
<td>Copie de valeurs</td>
</tr>
<tr>
<td align="right">3</td>
<td><code>Indices_actions!B6:O18</code></td>
<td><code>Actions!L10:Y22</code></td>
<td>Copie de valeurs</td>
</tr>
<tr>
<td align="right">4</td>
<td><code>Immeubles!B6:M18</code></td>
<td><code>Immeubles!N12:Y24</code></td>
<td>Copie de valeurs</td>
</tr>
<tr>
<td align="right">7</td>
<td><code>Inflation___taux</code>, lignes 5 et 18 à partir de G</td>
<td><code>Spread!C1...</code> et <code>Spread!C2...</code></td>
<td>Les valeurs sont copiées horizontalement</td>
</tr>
</tbody>
</table><p>Le code contient aussi les étapes 5, 6, 8 et 9. Le détail exact de leurs adresses figure dans l’annexe du code VBA intégral ci-dessous.</p>
<h3 id="fonctions-associées">Fonctions associées</h3>
<ul>
<li><code>CopierPlage</code> : vérifie l’existence des feuilles et la compatibilité des dimensions, puis copie les données.</li>
<li><code>TraiterSpread</code> : copie les lignes 5 et 18 de <code>Inflation___taux</code> vers les deux premières lignes de <code>Spread</code>, à partir de <code>C1</code> et <code>C2</code>.</li>
<li><code>AppliquerModificationsCibles</code> : applique des valeurs et formules de contrôle dans différentes feuilles cibles, notamment à partir du paramètre annuel de la feuille <code>param</code>.</li>
</ul>
<h2 id="macro-debuggeur_1h_1s">5. Macro <code>debuggeur_1h_1s</code></h2>
<h3 id="finalité-2">Finalité</h3>
<p>Fournir une variante de diagnostic du traitement <code>1h-1s</code>.</p>
<h3 id="particularités">Particularités</h3>
<ul>
<li>Reprend l’essentiel des étapes de copie de la macro principale.</li>
<li>Produit des messages plus détaillés pour identifier l’étape en échec.</li>
<li>Ne constitue pas une fonction d’aide privée : elle est directement exécutable depuis Excel.</li>
<li>Le module contient ses propres versions de <code>TraiterSpread</code> et <code>CopierPlage</code>.</li>
</ul>
<h2 id="macro-mettreajourtouteslesliaisons">6. Macro <code>MettreAJourToutesLesLiaisons</code></h2>
<h3 id="finalité-3">Finalité</h3>
<p>Rafraîchir d’abord les fichiers sources, puis mettre à jour les liaisons Excel des fichiers dépendants.</p>
<h3 id="ordre-opérationnel">Ordre opérationnel</h3>
<ol>
<li>Désactive les alertes de mise à jour de liens, l’affichage écran et les alertes Excel.</li>
<li>Lit la feuille <code>param</code>.</li>
<li><strong>Phase sources :</strong> parcourt la colonne C et appelle <code>OuvrirEtRafraichirSource</code>.</li>
<li><strong>Phase dépendants :</strong> parcourt la colonne A et appelle <code>OuvrirEtMettreAJourLien</code>.</li>
<li>Produit un rapport de succès et d’échecs.</li>
<li>Restaure les paramètres Excel, y compris en cas d’erreur gérée.</li>
</ol>
<h3 id="fonctions-appelées-1">Fonctions appelées</h3>
<h4 id="ouvriretrafraichirsource"><code>OuvrirEtRafraichirSource</code></h4>
<ul>
<li>Recherche le fichier avec les extensions <code>""</code>, <code>.xlsm</code>, <code>.xlsx</code>, puis <code>.xls</code>.</li>
<li>Ouvre le classeur avec <code>UpdateLinks:=0</code>.</li>
<li>Lance <code>RefreshAll</code>.</li>
<li>Force les calculs Excel.</li>
<li>Sauvegarde, puis ferme le classeur.</li>
</ul>
<h4 id="ouvriretmettreajourlien"><code>OuvrirEtMettreAJourLien</code></h4>
<ul>
<li>Recherche le fichier avec les mêmes extensions.</li>
<li>Ouvre le classeur sans mise à jour automatique à l’ouverture.</li>
<li>Exécute <code>UpdateLink Type:=xlExcelLinks</code>.</li>
<li>Force la sauvegarde afin d’enregistrer les nouvelles valeurs et de mettre à jour l’horodatage du fichier.</li>
<li>Ferme le classeur.</li>
</ul>
<h2 id="gestion-des-erreurs-et-performance">7. Gestion des erreurs et performance</h2>
<h3 id="mécanismes-présents">Mécanismes présents</h3>
<ul>
<li>Contrôle des paramètres obligatoires.</li>
<li>Ouverture des fichiers sous <code>On Error Resume Next</code>, suivie d’un contrôle de l’objet <code>Workbook</code>.</li>
<li>Compteurs de fichiers réussis et échoués.</li>
<li>Fonctions booléennes pour remonter le succès ou l’échec d’un sous-traitement.</li>
<li>Restauration des paramètres Excel dans les sorties normales et plusieurs gestionnaires d’erreurs.</li>
</ul>
<h3 id="points-de-vigilance">Points de vigilance</h3>
<ol>
<li><strong>Noms publics dupliqués :</strong> <code>TraiterSpread</code> et <code>CopierPlage</code> existent à la fois dans <code>Module_Traitement_1H_1R</code> et <code>Module1</code>. Selon l’état du projet, cela peut provoquer une erreur VBA <strong>« Nom ambigu détecté »</strong> ou rendre les appels non qualifiés difficiles à maintenir.</li>
<li><strong>Usage étendu de <code>On Error Resume Next</code> :</strong> certaines causes précises peuvent être masquées; le rapport indique alors l’échec sans toujours fournir l’erreur technique complète.</li>
<li><strong>Sauvegarde automatique :</strong> les macros modifient définitivement les fichiers cibles et les fichiers liés. Une copie de sauvegarde est recommandée avant exécution.</li>
<li><strong>Dépendance aux noms et adresses fixes :</strong> toute modification d’un nom d’onglet, d’une plage ou de la structure des maquettes peut bloquer une étape.</li>
<li><strong>Rafraîchissements asynchrones :</strong> <code>RefreshAll</code> peut lancer certains connecteurs en arrière-plan. Le code ne contient pas d’attente explicite de fin de toutes les requêtes avant sauvegarde.</li>
<li><strong>Commentaire incohérent :</strong> le code annonce « 8 traitements » mais déclare et contrôle jusqu’à <code>ok9</code>.</li>
<li><strong>Absence de journal persistant :</strong> le compte rendu est affiché dans une boîte de dialogue, sans écriture dans une feuille de log ou un fichier texte.</li>
</ol>
<h2 id="recommandations-d’amélioration">8. Recommandations d’amélioration</h2>
<ul>
<li>Renommer ou rendre <code>Private</code> les fonctions dupliquées du module de débogage.</li>
<li>Centraliser les mappages source-cible dans une table de paramétrage plutôt que dans le code.</li>
<li>Remplacer les blocs généraux <code>On Error Resume Next</code> par des gestionnaires d’erreurs ciblés.</li>
<li>Ajouter un journal portant le fichier, l’étape, l’heure, le statut et le message d’erreur.</li>
<li>Vérifier le statut <code>ReadOnly</code> avant toute sauvegarde.</li>
<li>Attendre explicitement la fin des connexions et requêtes asynchrones avant de sauvegarder.</li>
<li>Ajouter un mode « simulation » sans sauvegarde pour tester les chemins et les feuilles.</li>
<li>Sauvegarder une copie horodatée des fichiers avant leur modification en production.</li>
</ul>
<h2 id="procédures-recensées">9. Procédures recensées</h2>

<table>
<thead>
<tr>
<th>Module</th>
<th>Procédure</th>
<th>Type</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>Module_traitement_31h_31s</code></td>
<td><code>CopierOngletsVersFichiersCibles_31h_31s</code></td>
<td>Macro principale</td>
</tr>
<tr>
<td><code>Module_traitement_31h_31s</code></td>
<td><code>CopierOngletComplet</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module_traitement_31h_31s</code></td>
<td><code>CopierPlageEnteteClassique</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module_traitement_31h_31s</code></td>
<td><code>CopierPlageEnteteInflation</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module_Traitement_1H_1R</code></td>
<td><code>CopierDonneesVersFichiersCibles_1h_1s</code></td>
<td>Macro principale</td>
</tr>
<tr>
<td><code>Module_Traitement_1H_1R</code></td>
<td><code>TraiterSpread</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module_Traitement_1H_1R</code></td>
<td><code>AppliquerModificationsCibles</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module_Traitement_1H_1R</code></td>
<td><code>CopierPlage</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module1</code></td>
<td><code>debuggeur_1h_1s</code></td>
<td>Macro de diagnostic</td>
</tr>
<tr>
<td><code>Module1</code></td>
<td><code>TraiterSpread</code></td>
<td>Fonction dupliquée</td>
</tr>
<tr>
<td><code>Module1</code></td>
<td><code>CopierPlage</code></td>
<td>Fonction dupliquée</td>
</tr>
<tr>
<td><code>Module2</code></td>
<td><code>MettreAJourToutesLesLiaisons</code></td>
<td>Macro principale</td>
</tr>
<tr>
<td><code>Module2</code></td>
<td><code>OuvrirEtRafraichirSource</code></td>
<td>Fonction utilitaire</td>
</tr>
<tr>
<td><code>Module2</code></td>
<td><code>OuvrirEtMettreAJourLien</code></td>
<td>Fonction utilitaire</td>
</tr>
</tbody>
</table><h2 id="annexe-a.-code-vba-intégral-extrait">Annexe A. Code VBA intégral extrait</h2>
<blockquote>
<p>Cette annexe retranscrit le code présent dans le projet VBA afin de permettre l’audit, la maintenance et la comparaison avec les futures versions.</p>
</blockquote>
<pre class=" language-vb"><code class="prism  language-vb">VBA MACRO ThisWorkbook.cls 
in file: xl/vbaProject.bin - OLE stream: 'VBA/ThisWorkbook'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Feuil1.cls 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Feuil1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Module_traitement_31h_31s.bas 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Module_traitement_31h_31s'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Option Explicit

Sub CopierOngletsVersFichiersCibles_31h_31s()
    Dim wbLanceur As Workbook
    Dim wsParam As Worksheet
    Dim sourceFileName As String
    Dim sourceWorkbook As Workbook
    Dim targetWorkbook As Workbook
    Dim targetFile As String
    Dim lastRow As Long
    Dim listRange As Range
    Dim cell As Range
    Dim startTime As Date
    
    startTime = Now
    
    Set wbLanceur = ThisWorkbook
    Set wsParam = wbLanceur.Worksheets("param")
    sourceFileName = wsParam.Range("B2").Value
    
    If sourceFileName = "" Then
        MsgBox "La cellule B2 de l'onglet 'param' est vide.", vbCritical
        Exit Sub
    End If
    
    lastRow = wsParam.Cells(wsParam.Rows.Count, "A").End(xlUp).Row
    If lastRow &lt; 2 Then
        MsgBox "Aucun fichier cible listé dans la colonne A (à partir de A2).", vbCritical
        Exit Sub
    End If
    Set listRange = wsParam.Range("A2:A" &amp; lastRow)
    
    ' Désactiver les fonctionnalités AVANT d'ouvrir le classeur source
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    Application.Calculation = xlCalculationManual
    Application.EnableEvents = False
    Application.CalculateBeforeSave = False
    
    On Error Resume Next
    Set sourceWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; sourceFileName, UpdateLinks:=0)
    On Error GoTo 0
    
    If sourceWorkbook Is Nothing Then
        MsgBox "Impossible d'ouvrir le fichier source : " &amp; sourceFileName, vbCritical
        ' Réactiver avant de sortir
        Application.Calculation = xlCalculationAutomatic
        Application.EnableEvents = True
        Application.DisplayAlerts = True
        Application.ScreenUpdating = True
        Exit Sub
    End If
    
    Dim okCount As Long, errorCount As Long
    okCount = 0
    errorCount = 0
    
    For Each cell In listRange
        targetFile = Trim(cell.Value)
        If targetFile &lt;&gt; "" Then
            On Error Resume Next
            Set targetWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; targetFile, UpdateLinks:=0)
            On Error GoTo 0
            
            If targetWorkbook Is Nothing Then
                errorCount = errorCount + 1
            Else
                Dim ok1 As Boolean, ok2 As Boolean, ok3 As Boolean, ok4 As Boolean
                ok1 = CopierOngletComplet(sourceWorkbook, targetWorkbook, "Courbes_taux")
                ok2 = CopierOngletComplet(sourceWorkbook, targetWorkbook, "Spreads_courbes_taux")
                ok3 = CopierOngletComplet(sourceWorkbook, targetWorkbook, "Spreads_de_change")
                ok4 = CopierOngletComplet(sourceWorkbook, targetWorkbook, "Indices_change")
                
                Dim ok5 As Boolean, ok6 As Boolean, ok7 As Boolean
                ok5 = CopierPlageEnteteInflation(sourceWorkbook, targetWorkbook)
                ok6 = CopierPlageEnteteClassique(sourceWorkbook, targetWorkbook, "Indices_actions")
                ok7 = CopierPlageEnteteClassique(sourceWorkbook, targetWorkbook, "Immeubles")
                
                If ok1 And ok2 And ok3 And ok4 And ok5 And ok6 And ok7 Then
                    okCount = okCount + 1
                Else
                    errorCount = errorCount + 1
                End If
                
                targetWorkbook.Save
                targetWorkbook.Close SaveChanges:=False
            End If
        End If
    Next cell
    
    sourceWorkbook.Close SaveChanges:=False
    
    Application.Calculation = xlCalculationAutomatic
    Application.EnableEvents = True
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    
    Dim duration As String
    duration = Format(Now - startTime, "hh:mm:ss")
    
    Dim logMsg As String
    logMsg = "Traitement terminé en " &amp; duration &amp; vbCrLf &amp; _
             "Fichiers traités : " &amp; (okCount + errorCount) &amp; vbCrLf &amp; _
             "? Succès : " &amp; okCount &amp; vbCrLf &amp; _
             "? Erreurs : " &amp; errorCount
    
    If errorCount &gt; 0 Then
        logMsg = logMsg &amp; vbCrLf &amp; vbCrLf &amp; "?? Vérifiez les fichiers en erreur (onglets manquants ou structure différente)."
    End If
    
    MsgBox logMsg, vbInformation, "Rapport de copie"
End Sub

' ============================================================================
' FONCTION 1 : Copie INTÉGRALE d'un onglet (optimisée)
' ============================================================================
Function CopierOngletComplet(sourceWB As Workbook, targetWB As Workbook, sheetName As String) As Boolean
    On Error GoTo ErrorHandler
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    
    Set wsSrc = sourceWB.Worksheets(sheetName)
    Set wsTgt = targetWB.Worksheets(sheetName)
    
    ' Nettoyer uniquement la zone utilisée de la feuille cible
    If Not wsTgt.UsedRange Is Nothing Then
        wsTgt.UsedRange.Clear
    End If
    
    ' Copier uniquement la zone utilisée de la source
    wsSrc.UsedRange.Copy Destination:=wsTgt.Range("A1")
    Application.CutCopyMode = False
    
    CopierOngletComplet = True
    Exit Function
    
ErrorHandler:
    CopierOngletComplet = False
End Function

' ============================================================================
' FONCTION 2 : Copie PARTIELLE CLASSIQUE (optimisée)
' ============================================================================
Function CopierPlageEnteteClassique(sourceWB As Workbook, targetWB As Workbook, sheetName As String) As Boolean
    On Error GoTo ErrorHandler
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    Dim lastCol As Long
    Dim lastColTgt As Long
    Dim lastRowA As Long
    
    Set wsSrc = sourceWB.Worksheets(sheetName)
    Set wsTgt = targetWB.Worksheets(sheetName)
    
    ' --- NETTOYAGE CIBLE (limité aux zones utilisées) ---
    ' Colonne A : effacer seulement jusqu'à la dernière ligne utilisée
    lastRowA = wsTgt.Cells(wsTgt.Rows.Count, "A").End(xlUp).Row
    If lastRowA &gt;= 1 Then
        wsTgt.Range("A1:A" &amp; lastRowA).ClearContents ' ClearContents suffit, les formats seront recopiés
    End If
    
    ' Lignes 1 à 18, colonnes B à dernière colonne utilisée du destinataire
    lastColTgt = wsTgt.Cells(18, wsTgt.Columns.Count).End(xlToLeft).Column
    If lastColTgt &gt; 1 Then
        wsTgt.Range(wsTgt.Cells(1, 2), wsTgt.Cells(18, lastColTgt)).ClearContents
    End If
    
    ' --- COPIE ---
    ' Colonne A entière (limitée à la plage utilisée de la source)
    Dim lastRowSrcA As Long
    lastRowSrcA = wsSrc.Cells(wsSrc.Rows.Count, "A").End(xlUp).Row
    If lastRowSrcA &gt;= 1 Then
        wsSrc.Range("A1:A" &amp; lastRowSrcA).Copy Destination:=wsTgt.Range("A1")
        Application.CutCopyMode = False
    End If
    
    ' Lignes 1 à 18, colonnes B à dernière colonne source
    lastCol = wsSrc.Cells(18, wsSrc.Columns.Count).End(xlToLeft).Column
    If lastCol &gt; 1 Then
        wsSrc.Range(wsSrc.Cells(1, 2), wsSrc.Cells(18, lastCol)).Copy Destination:=wsTgt.Range("B1")
        Application.CutCopyMode = False
    End If
    
    CopierPlageEnteteClassique = True
    Exit Function
    
ErrorHandler:
    CopierPlageEnteteClassique = False
End Function

' ============================================================================
' FONCTION 3 : Copie SPÉCIFIQUE pour Inflation___taux (optimisée)
' ============================================================================
Function CopierPlageEnteteInflation(sourceWB As Workbook, targetWB As Workbook) As Boolean
    On Error GoTo ErrorHandler
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    Dim sheetName As String
    Dim lastColSrc As Long
    Dim lastRowSrc As Long
    Dim lastColTgt As Long
    Dim lastRowTgt As Long
    
    sheetName = "Inflation___taux"
    Set wsSrc = sourceWB.Worksheets(sheetName)
    Set wsTgt = targetWB.Worksheets(sheetName)
    
    ' --- DÉTERMINER LES LIMITES ---
    lastColSrc = wsSrc.Cells(18, wsSrc.Columns.Count).End(xlToLeft).Column
    If lastColSrc &lt; 6 Then lastColSrc = 6
    lastRowSrc = wsSrc.Cells(wsSrc.Rows.Count, "F").End(xlUp).Row
    If lastRowSrc &lt; 19 Then lastRowSrc = 19
    
    lastColTgt = wsTgt.Cells(19, wsTgt.Columns.Count).End(xlToLeft).Column
    If lastColTgt &lt; 6 Then lastColTgt = 6
    lastRowTgt = wsTgt.Cells(wsTgt.Rows.Count, "F").End(xlUp).Row
    If lastRowTgt &lt; 19 Then lastRowTgt = 19
    
    ' --- NETTOYAGE CIBLE (uniquement les zones à remplacer ou excédentaires) ---
    ' Colonne A : effacer jusqu'à la dernière ligne utilisée
    Dim lastRowTgtA As Long
    lastRowTgtA = wsTgt.Cells(wsTgt.Rows.Count, "A").End(xlUp).Row
    If lastRowTgtA &gt;= 1 Then
        wsTgt.Range("A1:A" &amp; lastRowTgtA).ClearContents
    End If
    
    ' Lignes 1 à 18, colonnes B à lastColTgt (pour enlever les anciens en-têtes)
    If lastColTgt &gt; 1 Then
        wsTgt.Range(wsTgt.Cells(1, 2), wsTgt.Cells(18, lastColTgt)).ClearContents
    End If
    
    ' Zone F19:LastRowTgt x LastColTgt (valeurs résiduelles)
    If lastRowTgt &gt;= 19 And lastColTgt &gt;= 6 Then
        wsTgt.Range(wsTgt.Cells(19, 6), wsTgt.Cells(lastRowTgt, lastColTgt)).ClearContents
    End If
    
    ' --- COPIE DES DONNÉES ---
    ' Colonne A source (limitée à la plage utilisée)
    Dim lastRowSrcA As Long
    lastRowSrcA = wsSrc.Cells(wsSrc.Rows.Count, "A").End(xlUp).Row
    If lastRowSrcA &gt;= 1 Then
        wsSrc.Range("A1:A" &amp; lastRowSrcA).Copy Destination:=wsTgt.Range("A1")
        Application.CutCopyMode = False
    End If
    
    ' Lignes 1 à 18, colonnes B à lastColSrc
    If lastColSrc &gt; 1 Then
        wsSrc.Range(wsSrc.Cells(1, 2), wsSrc.Cells(18, lastColSrc)).Copy Destination:=wsTgt.Range("B1")
        Application.CutCopyMode = False
    End If
    
    ' Duplication ligne 18 sur F19:LastRowSrc x LastColSrc
    If lastRowSrc &gt;= 19 And lastColSrc &gt;= 6 Then
        Dim sourceRow As Range
        Set sourceRow = wsSrc.Range(wsSrc.Cells(18, 6), wsSrc.Cells(18, lastColSrc))
        sourceRow.Copy Destination:=wsTgt.Range(wsTgt.Cells(19, 6), wsTgt.Cells(lastRowSrc, lastColSrc))
        Application.CutCopyMode = False
    End If
    
    CopierPlageEnteteInflation = True
    Exit Function
    
ErrorHandler:
    CopierPlageEnteteInflation = False
End Function

-------------------------------------------------------------------------------
VBA MACRO Module_Traitement_1H_1R.bas 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Module_Traitement_1H_1R'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Option Explicit

Sub CopierDonneesVersFichiersCibles_1h_1s()
    Dim wbLanceur As Workbook
    Dim wsParam As Worksheet
    Dim sourceFileName As String
    Dim sourceWorkbook As Workbook
    Dim targetWorkbook As Workbook
    Dim targetFile As String
    Dim lastRow As Long
    Dim listRange As Range
    Dim cell As Range
    Dim startTime As Date
    
    startTime = Now
    Set wbLanceur = ThisWorkbook
    Set wsParam = wbLanceur.Worksheets("param")
    sourceFileName = wsParam.Range("B2").Value
    
    If sourceFileName = "" Then
        MsgBox "La cellule B2 de l'onglet 'param' est vide.", vbCritical
        Exit Sub
    End If
    
    lastRow = wsParam.Cells(wsParam.Rows.Count, "C").End(xlUp).Row
    If lastRow &lt; 2 Then
        MsgBox "Aucun fichier cible listé dans la colonne C (à partir de C2).", vbCritical
        Exit Sub
    End If
    Set listRange = wsParam.Range("C2:C" &amp; lastRow)
    
    On Error Resume Next
    Set sourceWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; sourceFileName, UpdateLinks:=0)
    On Error GoTo 0
    
    If sourceWorkbook Is Nothing Then
        MsgBox "Impossible d'ouvrir le fichier source : " &amp; sourceFileName, vbCritical
        Exit Sub
    End If
    
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    
    Dim okCount As Long, errorCount As Long
    okCount = 0
    errorCount = 0
    
    For Each cell In listRange
        targetFile = Trim(cell.Value)
        If targetFile &lt;&gt; "" Then
            On Error Resume Next
            Set targetWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; targetFile, UpdateLinks:=0)
            On Error GoTo 0
            
            If targetWorkbook Is Nothing Then
                errorCount = errorCount + 1
            Else
                ' ----------------------------------------------------------------
                ' EXÉCUTION DES 8 TRAITEMENTS
                ' ----------------------------------------------------------------
                Dim ok1 As Boolean, ok2 As Boolean, ok3 As Boolean, ok4 As Boolean
                Dim ok5 As Boolean, ok6 As Boolean, ok7 As Boolean, ok8 As Boolean, ok9 As Boolean
                Dim errMsg As String
                errMsg = ""
                
                ' Étape 1 : A4:A630 -&gt; Taux I9:I635
                ok1 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "A4:A630", _
                                  "Taux", "I9:I635", True)
                If Not ok1 Then errMsg = errMsg &amp; " [Étape1: Taux I9]"
                
                ' Étape 2 : D6:E18 -&gt; Taux J11:K23
                ok2 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "D6:E18", _
                                  "Taux", "J11:K23", False)
                If Not ok2 Then errMsg = errMsg &amp; " [Étape2: Taux J11]"
                
                ' Étape 3 : Indices_actions B6:O18 -&gt; Actions L10:Y22
                ok3 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Indices_actions", "B6:O18", _
                                  "Actions", "L10:Y22", False)
                If Not ok3 Then errMsg = errMsg &amp; " [Étape3: Actions]"
                
                ' Étape 4 : Immeubles B6:M18 -&gt; Immeubles N12:Y24
                ok4 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Immeubles", "B6:M18", _
                                  "Immeubles", "N12:Y24", False)
                If Not ok4 Then errMsg = errMsg &amp; " [Étape4: Immeubles]"
                
                ' Étape 5 : Inflation___taux C6:C18 -&gt; Croissance C10:C22
                ok5 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "C6:C18", _
                                  "Croissance", "C10:C22", False)
                If Not ok5 Then errMsg = errMsg &amp; " [Étape5: Croissance]"
                
                ' Étape 6 : Inflation___taux B6:B18 -&gt; Inflation C10:C22
                ok6 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "B6:B18", _
                                  "Inflation", "C10:C22", False)
                If Not ok6 Then errMsg = errMsg &amp; " [Étape6: Inflation]"
                
                ' Étape 7 : Traitement Spread (effacement lignes 1-2, collages C1/C2, G7:G47, colonne A en G7, duplication I..S)
                ok7 = TraiterSpread(sourceWorkbook, targetWorkbook)
                If Not ok7 Then errMsg = errMsg &amp; " [Étape7: Spread]"
                
                ' Étape 8 : Nouveaux traitements (cellule nommée "n", formules Immeubles/Actions, effacements)
                ok8 = AppliquerModificationsCibles(sourceWorkbook, targetWorkbook)
                If Not ok8 Then errMsg = errMsg &amp; " [Étape8: Modifications finales]"
                ' Étape 9 : Copie F19:F630 (source) -&gt; R25:R636 (cible) dans Immeubles
                ok9 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Immeubles", "F19:F630", _
                                  "Immeubles", "R25:R636", False)
                If Not ok9 Then errMsg = errMsg &amp; " [Étape9: Immeubles R25]"
                
                ' ----------------------------------------------------------------
                ' BILAN POUR CE FICHIER
                ' ----------------------------------------------------------------
                If ok1 And ok2 And ok3 And ok4 And ok5 And ok6 And ok7 And ok8 And ok9 Then
                    okCount = okCount + 1
                Else
                    errorCount = errorCount + 1
                    MsgBox "? Échec sur le fichier : " &amp; targetFile &amp; vbCrLf &amp; _
                           "Étapes en erreur :" &amp; errMsg &amp; vbCrLf &amp; vbCrLf &amp; _
                           "Vérifiez les noms des onglets, des plages et du nom défini 'n'.", _
                           vbCritical, "Débogage"
                End If
                
                targetWorkbook.Save
                targetWorkbook.Close SaveChanges:=False
            End If
        End If
    Next cell
    
    sourceWorkbook.Close SaveChanges:=False
    
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    
    Dim duration As String
    duration = Format(Now - startTime, "hh:mm:ss")
    
    Dim logMsg As String
    logMsg = "Traitement terminé en " &amp; duration &amp; vbCrLf &amp; _
             "Fichiers traités : " &amp; (okCount + errorCount) &amp; vbCrLf &amp; _
             "? Succès : " &amp; okCount &amp; vbCrLf &amp; _
             "? Erreurs : " &amp; errorCount
    
    If errorCount &gt; 0 Then
        logMsg = logMsg &amp; vbCrLf &amp; vbCrLf &amp; "?? Des messages d'erreur détaillés se sont affichés pendant l'exécution."
    End If
    
    MsgBox logMsg, vbInformation, "Rapport de copie"
End Sub


' ============================================================================
' FONCTION : Traitement spécifique de l'onglet "Spread" (conservée)
' ============================================================================
Function TraiterSpread(sourceWB As Workbook, targetWB As Workbook) As Boolean
    On Error GoTo ErrorHandler
    
    Dim wsSpreadSrc As Worksheet
    Dim wsSpreadTgt As Worksheet
    Dim wsCourbesSrc As Worksheet
    
    Dim lastCol5 As Long
    Dim lastCol18 As Long
    Dim rngSrc5 As Range
    Dim rngSrc18 As Range
    
    Set wsSpreadSrc = sourceWB.Worksheets("Inflation___taux")
    Set wsSpreadTgt = targetWB.Worksheets("Spread")
    
    ' 1) Effacer lignes 1 et 2
    wsSpreadTgt.Rows("1:2").ClearContents
    
    ' 2) Coller ligne 5 du source (G5:dernière colonne) en C1
    lastCol5 = wsSpreadSrc.Cells(5, wsSpreadSrc.Columns.Count).End(xlToLeft).Column
    If lastCol5 &gt;= 7 Then
        Set rngSrc5 = wsSpreadSrc.Range(wsSpreadSrc.Cells(5, 7), wsSpreadSrc.Cells(5, lastCol5))
        wsSpreadTgt.Range("C1").Resize(1, rngSrc5.Columns.Count).Value = rngSrc5.Value
    End If
    
    ' 3) Coller ligne 18 du source (G18:dernière colonne) en C2
    lastCol18 = wsSpreadSrc.Cells(18, wsSpreadSrc.Columns.Count).End(xlToLeft).Column
    If lastCol18 &gt;= 7 Then
        Set rngSrc18 = wsSpreadSrc.Range(wsSpreadSrc.Cells(18, 7), wsSpreadSrc.Cells(18, lastCol18))
        wsSpreadTgt.Range("C2").Resize(1, rngSrc18.Columns.Count).Value = rngSrc18.Value
    End If
    
    ' 4) Effacer G7:G47
    wsSpreadTgt.Range("G7:G47").ClearContents
    
    ' 5) Onglet "Spreads_courbes_taux" du source
    Set wsCourbesSrc = sourceWB.Worksheets("Spreads_courbes_taux")
    
    ' 6) Copier colonne A (A6:dernière ligne) en G7
    Dim lastRowA As Long
    lastRowA = wsCourbesSrc.Cells(wsCourbesSrc.Rows.Count, "A").End(xlUp).Row
    If lastRowA &gt;= 6 Then
        Dim rngColA As Range
        Set rngColA = wsCourbesSrc.Range("A6:A" &amp; lastRowA)
        wsSpreadTgt.Range("G7").Resize(rngColA.Rows.Count, 1).Value = rngColA.Value
    End If
    
    ' 7) CAS B : Dupliquer la dernière colonne de la ligne 6 dans les colonnes I à S
    Dim lastColCourbes As Long
    Dim lastRowCourbes As Long
    Dim rngColDerniere As Range
    Dim colOffset As Long
    Dim nbCols As Long
    
    lastColCourbes = wsCourbesSrc.Cells(6, wsCourbesSrc.Columns.Count).End(xlToLeft).Column
    If lastColCourbes &gt;= 1 Then
        lastRowCourbes = wsCourbesSrc.Cells(wsCourbesSrc.Rows.Count, lastColCourbes).End(xlUp).Row
        If lastRowCourbes &gt;= 6 Then
            Set rngColDerniere = wsCourbesSrc.Range(wsCourbesSrc.Cells(6, lastColCourbes), _
                                                    wsCourbesSrc.Cells(lastRowCourbes, lastColCourbes))
            nbCols = 11 ' I à S = 11 colonnes
            For colOffset = 0 To nbCols - 1
                wsSpreadTgt.Range("I7").Offset(0, colOffset).Resize(rngColDerniere.Rows.Count, 1).Value = rngColDerniere.Value
            Next colOffset
        End If
    End If
    
    TraiterSpread = True
    Exit Function
    
ErrorHandler:
    TraiterSpread = False
End Function


' ============================================================================
' FONCTION : Nouveaux traitements (version DÉBOGAGE)
' ============================================================================
Function AppliquerModificationsCibles(sourceWB As Workbook, targetWB As Workbook) As Boolean
    On Error GoTo ErrorHandler
    
    Dim wsSpread As Worksheet
    Dim wsImmeubles As Worksheet
    Dim wsActions As Worksheet
    Dim wsSynth As Worksheet
    Dim valSource As Variant
    Dim i As Long
    
    ' ------------------------------------------------------------
    ' ÉTAPE 1 : Lire la valeur dans I2 de l'onglet "parma" (source)
    ' ------------------------------------------------------------
    On Error Resume Next
    valSource = ThisWorkbook.Worksheets("param").Range("I2").Value
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.1 : Impossible de lire 'parma!I2' dans le fichier source." &amp; vbCrLf &amp; _
               "Vérifiez que l'onglet s'appelle bien 'parma' (et non 'param').", vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    
    ' ------------------------------------------------------------
    ' ÉTAPE 2 : Assigner la valeur au nom "n" dans le fichier cible
    ' ------------------------------------------------------------
    On Error Resume Next
    targetWB.Names("n").RefersToRange.Value = valSource
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.2 : Impossible d'écrire dans la cellule nommée 'n' du fichier " &amp; targetWB.Name &amp; vbCrLf &amp; _
               "Le nom 'n' est-il défini dans ce fichier ? (Formules &gt; Gestionnaire de noms)", vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    
    ' ------------------------------------------------------------
    ' ÉTAPE 3 : Supprimer A5:C50 dans l'onglet Spread
    ' ------------------------------------------------------------
    On Error Resume Next
    Set wsSpread = targetWB.Worksheets("Spread")
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.3 : L'onglet 'Spread' est introuvable dans " &amp; targetWB.Name, vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    wsSpread.Range("A5:C50").ClearContents
    
    ' ------------------------------------------------------------
    ' ÉTAPE 4 : Formules dans l'onglet Immeubles (I, J, K, L)
    ' ------------------------------------------------------------
    On Error Resume Next
    Set wsImmeubles = targetWB.Worksheets("Immeubles")
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.4 : L'onglet 'Immeubles' est introuvable dans " &amp; targetWB.Name, vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    
    For i = 12 To 24
        wsImmeubles.Range("I" &amp; i).Formula = "=N" &amp; i
        wsImmeubles.Range("J" &amp; i).Formula = "=Q" &amp; i
        wsImmeubles.Range("K" &amp; i).Formula = "=T" &amp; i
        wsImmeubles.Range("L" &amp; i).Formula = "=W" &amp; i
    Next i
    
    ' ------------------------------------------------------------
    ' ÉTAPE 5 : Formules dans l'onglet Actions (G, H, I, J)
    ' ------------------------------------------------------------
    On Error Resume Next
    Set wsActions = targetWB.Worksheets("Actions")
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.5 : L'onglet 'Actions' est introuvable dans " &amp; targetWB.Name, vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    
    For i = 10 To 22
        wsActions.Range("G" &amp; i).Formula = "=L" &amp; i
        wsActions.Range("H" &amp; i).Formula = "=O" &amp; i
        wsActions.Range("I" &amp; i).Formula = "=M" &amp; i
        wsActions.Range("J" &amp; i).Formula = "=R" &amp; i
    Next i
    
    ' ------------------------------------------------------------
    ' ÉTAPE 6 : Effacer dans l'onglet Synthèse (B7:N31)
    ' ------------------------------------------------------------
    On Error Resume Next
    Set wsSynth = targetWB.Worksheets("Synthèse")
    If Err.Number &lt;&gt; 0 Then
        MsgBox "? Erreur Étape 8.6 : L'onglet 'Synthèse' est introuvable dans " &amp; targetWB.Name, vbCritical
        Err.Clear
        AppliquerModificationsCibles = False
        Exit Function
    End If
    On Error GoTo 0
    wsSynth.Range("B31:N47").ClearContents   ' ?? Ajustez ici si la plage est différente
    
    ' ------------------------------------------------------------
    ' TOUT EST RÉUSSI
    ' ------------------------------------------------------------
    AppliquerModificationsCibles = True
    Exit Function
    
ErrorHandler:
    MsgBox "? Erreur inattendue dans l'étape 8 du fichier " &amp; targetWB.Name &amp; vbCrLf &amp; _
           "Erreur n° " &amp; Err.Number &amp; " : " &amp; Err.Description, vbCritical
    AppliquerModificationsCibles = False
End Function


' ============================================================================
' FONCTION : Copie d'une plage source vers une plage destination
' ============================================================================
Function CopierPlage(sourceWB As Workbook, targetWB As Workbook, _
                     sourceSheet As String, sourceAddr As String, _
                     targetSheet As String, targetAddr As String, _
                     copyFormat As Boolean) As Boolean
    On Error GoTo ErrorHandler
    
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    Dim rngSrc As Range
    Dim rngTgt As Range
    
    Set wsSrc = sourceWB.Worksheets(sourceSheet)
    Set wsTgt = targetWB.Worksheets(targetSheet)
    
    Set rngSrc = wsSrc.Range(sourceAddr)
    Set rngTgt = wsTgt.Range(targetAddr)
    
    If copyFormat Then
        rngSrc.Copy
        rngTgt.PasteSpecial Paste:=xlPasteValuesAndNumberFormats
        rngTgt.PasteSpecial Paste:=xlPasteFormats
        Application.CutCopyMode = False
    Else
        rngTgt.Value = rngSrc.Value
    End If
    
    CopierPlage = True
    Exit Function
    
ErrorHandler:
    CopierPlage = False
End Function




-------------------------------------------------------------------------------
VBA MACRO Module1.bas 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Module1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Option Explicit

Sub debuggeur_1h_1s()
    Dim wbLanceur As Workbook
    Dim wsParam As Worksheet
    Dim sourceFileName As String
    Dim sourceWorkbook As Workbook
    Dim targetWorkbook As Workbook
    Dim targetFile As String
    Dim lastRow As Long
    Dim listRange As Range
    Dim cell As Range
    Dim startTime As Date
    
    startTime = Now
    Set wbLanceur = ThisWorkbook
    Set wsParam = wbLanceur.Worksheets("param")
    sourceFileName = wsParam.Range("B2").Value
    
    If sourceFileName = "" Then
        MsgBox "La cellule B2 de l'onglet 'param' est vide.", vbCritical
        Exit Sub
    End If
    
    lastRow = wsParam.Cells(wsParam.Rows.Count, "C").End(xlUp).Row
    If lastRow &lt; 2 Then
        MsgBox "Aucun fichier cible listé dans la colonne C (à partir de C2).", vbCritical
        Exit Sub
    End If
    Set listRange = wsParam.Range("C2:C" &amp; lastRow)
    
    On Error Resume Next
    Set sourceWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; sourceFileName, UpdateLinks:=0)
    On Error GoTo 0
    
    If sourceWorkbook Is Nothing Then
        MsgBox "Impossible d'ouvrir le fichier source : " &amp; sourceFileName, vbCritical
        Exit Sub
    End If
    
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    
    Dim okCount As Long, errorCount As Long
    okCount = 0
    errorCount = 0
    
    For Each cell In listRange
        targetFile = Trim(cell.Value)
        If targetFile &lt;&gt; "" Then
            On Error Resume Next
            Set targetWorkbook = Workbooks.Open(Filename:=wbLanceur.Path &amp; "\" &amp; targetFile, UpdateLinks:=0)
            On Error GoTo 0
            
            If targetWorkbook Is Nothing Then
                errorCount = errorCount + 1
            Else
                ' ----------------------------------------------------------------
                ' EXÉCUTION DES 7 COPIES AVEC DÉBOGAGE PAS À PAS
                ' ----------------------------------------------------------------
                Dim ok1 As Boolean, ok2 As Boolean, ok3 As Boolean, ok4 As Boolean
                Dim ok5 As Boolean, ok6 As Boolean, ok7 As Boolean
                Dim errMsg As String
                errMsg = ""
                
                ' Étape 1
                ok1 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "A4:A630", _
                                  "Taux", "I9:I635", True)
                If Not ok1 Then errMsg = errMsg &amp; " [Étape1: Taux I9]"
                
                ' Étape 2
                ok2 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "D6:E18", _
                                  "Taux", "J11:K23", False)
                If Not ok2 Then errMsg = errMsg &amp; " [Étape2: Taux J11]"
                
                ' Étape 3
                ok3 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Indices_actions", "B6:O18", _
                                  "Actions", "L10:Y22", False)
                If Not ok3 Then errMsg = errMsg &amp; " [Étape3: Actions]"
                
                ' Étape 4
                ok4 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Immeubles", "B6:M18", _
                                  "Immeubles", "N12:Y24", False)
                If Not ok4 Then errMsg = errMsg &amp; " [Étape4: Immeubles]"
                
                ' Étape 5
                ok5 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "C6:C18", _
                                  "Croissance", "C10:C22", False)
                If Not ok5 Then errMsg = errMsg &amp; " [Étape5: Croissance]"
                
                ' Étape 6
                ok6 = CopierPlage(sourceWorkbook, targetWorkbook, _
                                  "Inflation___taux", "B6:B18", _
                                  "Inflation", "C10:C22", False)
                If Not ok6 Then errMsg = errMsg &amp; " [Étape6: Inflation]"
                
                ' Étape 7 (Spread)
                ok7 = TraiterSpread(sourceWorkbook, targetWorkbook)
                If Not ok7 Then errMsg = errMsg &amp; " [Étape7: Spread]"
                
                ' ----------------------------------------------------------------
                ' BILAN POUR CE FICHIER
                ' ----------------------------------------------------------------
                If ok1 And ok2 And ok3 And ok4 And ok5 And ok6 And ok7 Then
                    okCount = okCount + 1
                Else
                    errorCount = errorCount + 1
                    ' ?? AFFICHAGE DE L'ERREUR DÉTAILLÉE POUR CE FICHIER
                    MsgBox "? Échec sur le fichier : " &amp; targetFile &amp; vbCrLf &amp; _
                           "Étapes en erreur :" &amp; errMsg &amp; vbCrLf &amp; vbCrLf &amp; _
                           "Vérifiez les noms des onglets et des plages dans ce fichier.", _
                           vbCritical, "Débogage"
                End If
                
                targetWorkbook.Save
                targetWorkbook.Close SaveChanges:=False
            End If
        End If
    Next cell
    
    sourceWorkbook.Close SaveChanges:=False
    
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    
    Dim duration As String
    duration = Format(Now - startTime, "hh:mm:ss")
    
    Dim logMsg As String
    logMsg = "Traitement terminé en " &amp; duration &amp; vbCrLf &amp; _
             "Fichiers traités : " &amp; (okCount + errorCount) &amp; vbCrLf &amp; _
             "? Succès : " &amp; okCount &amp; vbCrLf &amp; _
             "? Erreurs : " &amp; errorCount
    
    If errorCount &gt; 0 Then
        logMsg = logMsg &amp; vbCrLf &amp; vbCrLf &amp; "?? Des messages d'erreur détaillés se sont affichés pendant l'exécution."
    End If
    
    MsgBox logMsg, vbInformation, "Rapport de copie"
End Sub


' ============================================================================
' FONCTION : Traitement Spread
' ============================================================================
Function TraiterSpread(sourceWB As Workbook, targetWB As Workbook) As Boolean
    On Error GoTo ErrorHandler
    
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    Dim lastCol5 As Long
    Dim lastCol18 As Long
    Dim rngSrc5 As Range
    Dim rngSrc18 As Range
    
    ' ?? Vérifiez ici que le nom de l'onglet est EXACTEMENT "Spread" (ou "Spreads")
    Set wsSrc = sourceWB.Worksheets("Inflation___taux")
    Set wsTgt = targetWB.Worksheets("Spread")
    
    ' Effacer lignes 1 et 2
    wsTgt.Rows("1:2").ClearContents
    
    ' Ligne 5
    lastCol5 = wsSrc.Cells(5, wsSrc.Columns.Count).End(xlToLeft).Column
    If lastCol5 &gt;= 7 Then
        Set rngSrc5 = wsSrc.Range(wsSrc.Cells(5, 7), wsSrc.Cells(5, lastCol5))
        wsTgt.Range("C1").Resize(1, rngSrc5.Columns.Count).Value = rngSrc5.Value
    End If
    
    ' Ligne 18
    lastCol18 = wsSrc.Cells(18, wsSrc.Columns.Count).End(xlToLeft).Column
    If lastCol18 &gt;= 7 Then
        Set rngSrc18 = wsSrc.Range(wsSrc.Cells(18, 7), wsSrc.Cells(18, lastCol18))
        wsTgt.Range("C2").Resize(1, rngSrc18.Columns.Count).Value = rngSrc18.Value
    End If
    
    TraiterSpread = True
    Exit Function
    
ErrorHandler:
    TraiterSpread = False
End Function


' ============================================================================
' FONCTION : Copie d'une plage
' ============================================================================
Function CopierPlage(sourceWB As Workbook, targetWB As Workbook, _
                     sourceSheet As String, sourceAddr As String, _
                     targetSheet As String, targetAddr As String, _
                     copyFormat As Boolean) As Boolean
    On Error GoTo ErrorHandler
    
    Dim wsSrc As Worksheet
    Dim wsTgt As Worksheet
    Dim rngSrc As Range
    Dim rngTgt As Range
    
    Set wsSrc = sourceWB.Worksheets(sourceSheet)
    Set wsTgt = targetWB.Worksheets(targetSheet)
    
    Set rngSrc = wsSrc.Range(sourceAddr)
    Set rngTgt = wsTgt.Range(targetAddr)
    
    If copyFormat Then
        rngSrc.Copy
        rngTgt.PasteSpecial Paste:=xlPasteValuesAndNumberFormats
        rngTgt.PasteSpecial Paste:=xlPasteFormats
        Application.CutCopyMode = False
    Else
        rngTgt.Value = rngSrc.Value
    End If
    
    CopierPlage = True
    Exit Function
    
ErrorHandler:
    CopierPlage = False
End Function

-------------------------------------------------------------------------------
VBA MACRO Module2.bas 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Module2'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Option Explicit

Sub MettreAJourToutesLesLiaisons()
    ' ------------------------------------------------------------------------
    ' ÉTAPE 1 : Ouvrir les sources (colonne C), rafraîchir, sauvegarder.
    ' ÉTAPE 2 : Ouvrir les dépendants (colonne A), mettre à jour les liens,
    '           puis FORCER LA SAUVEGARDE pour mettre à jour l'horodatage.
    ' ------------------------------------------------------------------------
    Dim ws As Worksheet
    Dim derniereLigneA As Long, derniereLigneC As Long
    Dim i As Long
    Dim nomFichier As String

    ' --- Variables pour le suivi ---
    Dim startTime As Double, endTime As Double
    Dim successA As Long, failA As Long   ' Compteurs pour colonne A
    Dim successC As Long, failC As Long   ' Compteurs pour colonne C
    Dim totalSeconds As Double
    Dim mins As Long, secs As Double
    Dim timeStr As String
    Dim logMsg As String

    ' --- Gestion d'erreur globale ---
    On Error GoTo ErrorHandler

    ' Désactiver les alertes et mises à jour d'écran pour accélérer
    Application.DisplayAlerts = False
    Application.ScreenUpdating = False
    Application.AskToUpdateLinks = False

    ' Référence à l'onglet "param"
    Set ws = ThisWorkbook.Worksheets("param")

    ' Dernières lignes non vides
    derniereLigneA = ws.Cells(ws.Rows.Count, 1).End(xlUp).Row
    derniereLigneC = ws.Cells(ws.Rows.Count, 3).End(xlUp).Row

    ' Initialiser les compteurs et le chronomètre
    successA = 0: failA = 0
    successC = 0: failC = 0
    startTime = Timer

    ' ========================================================================
    ' ÉTAPE 1 : TRAITEMENT DES FICHIERS SOURCES (Colonne C : 1hxxx, ..., 1sxxx)
    ' ========================================================================
    For i = 2 To derniereLigneC
        nomFichier = Trim(ws.Cells(i, 3).Value)
        If nomFichier &lt;&gt; "" Then
            If OuvrirEtRafraichirSource(nomFichier) Then
                successC = successC + 1
            Else
                failC = failC + 1
            End If
        End If
    Next i

    ' ========================================================================
    ' ÉTAPE 2 : TRAITEMENT DES FICHIERS DÉPENDANTS (Colonne A : 3_1hxxx, ..., 3_1sxxx)
    ' ========================================================================
    For i = 2 To derniereLigneA
        nomFichier = Trim(ws.Cells(i, 1).Value)
        If nomFichier &lt;&gt; "" Then
            If OuvrirEtMettreAJourLien(nomFichier) Then
                successA = successA + 1
            Else
                failA = failA + 1
            End If
        End If
    Next i

    ' ========================================================================
    ' AFFICHAGE DU RAPPORT FINAL
    ' ========================================================================
    endTime = Timer
    totalSeconds = endTime - startTime
    mins = Int(totalSeconds / 60)
    secs = totalSeconds - (mins * 60)

    If mins &gt; 0 Then
        timeStr = mins &amp; " minute" &amp; IIf(mins &gt; 1, "s", "") &amp; " et " &amp; Format(secs, "0.0") &amp; " seconde" &amp; IIf(secs &gt; 1, "s", "")
    Else
        timeStr = Format(secs, "0.0") &amp; " seconde" &amp; IIf(secs &gt; 1, "s", "")
    End If

    logMsg = "Traitement terminé en " &amp; timeStr &amp; vbCrLf &amp; vbCrLf &amp; _
             "--- SOURCES (1h... / 1s...) ---" &amp; vbCrLf &amp; _
             "Fichiers traités : " &amp; (successC + failC) &amp; vbCrLf &amp; _
             "? Succès : " &amp; successC &amp; vbCrLf &amp; _
             "? Erreurs : " &amp; failC &amp; vbCrLf &amp; vbCrLf &amp; _
             "--- DÉPENDANTS (3_1h... / 3_1s...) ---" &amp; vbCrLf &amp; _
             "Fichiers traités : " &amp; (successA + failA) &amp; vbCrLf &amp; _
             "? Succès : " &amp; successA &amp; vbCrLf &amp; _
             "? Erreurs : " &amp; failA

    MsgBox logMsg, vbInformation, "Rapport d'exécution"

    GoTo CleanExit

ErrorHandler:
    MsgBox "Une erreur inattendue est survenue : " &amp; Err.Description, vbCritical, "Erreur"

CleanExit:
    ' Restaurer les paramètres Excel
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    Application.AskToUpdateLinks = True
End Sub


' ========================================================================
' FONCTION POUR LES SOURCES (colonne C) : Ouvre, rafraîchit, sauvegarde.
' ========================================================================
Function OuvrirEtRafraichirSource(nomFichier As String) As Boolean
    Dim cheminComplet As String
    Dim wb As Workbook
    Dim extensions As Variant
    Dim i As Integer

    extensions = Array("", ".xlsm", ".xlsx", ".xls")
    On Error Resume Next

    For i = LBound(extensions) To UBound(extensions)
        cheminComplet = ThisWorkbook.Path &amp; "\" &amp; nomFichier &amp; extensions(i)

        If Dir(cheminComplet) &lt;&gt; "" Then
            Set wb = Workbooks.Open(cheminComplet, UpdateLinks:=0)

            If Not wb Is Nothing Then
                ' --- Rafraîchir TOUT (formules, TCD, Power Query, etc.) ---
                wb.RefreshAll
                Application.Calculate
                wb.Application.Calculate

                If Err.Number = 0 Then
                    ' --- SAUVEGARDER pour écrire les nouvelles valeurs sur le disque ---
                    wb.Save
                    OuvrirEtRafraichirSource = True
                Else
                    OuvrirEtRafraichirSource = False
                    Err.Clear
                End If

                wb.Close SaveChanges:=False
                Exit Function
            End If
        End If
    Next i

    OuvrirEtRafraichirSource = False
    On Error GoTo 0
End Function


' ========================================================================
' FONCTION POUR LES DÉPENDANTS (colonne A) :
' Ouvre, met à jour les liens, puis FORCE LA SAUVEGARDE (Solution 2).
' ========================================================================
Function OuvrirEtMettreAJourLien(nomFichier As String) As Boolean
    Dim cheminComplet As String
    Dim wb As Workbook
    Dim extensions As Variant
    Dim i As Integer

    extensions = Array("", ".xlsm", ".xlsx", ".xls")
    On Error Resume Next

    For i = LBound(extensions) To UBound(extensions)
        cheminComplet = ThisWorkbook.Path &amp; "\" &amp; nomFichier &amp; extensions(i)

        If Dir(cheminComplet) &lt;&gt; "" Then
            Set wb = Workbooks.Open(cheminComplet, UpdateLinks:=0)

            If Not wb Is Nothing Then
                ' Mise à jour de toutes les liaisons
                wb.UpdateLink Type:=xlExcelLinks

                If Err.Number = 0 Then
                    ' =============================================================
                    ' ?? SOLUTION 2 : FORCER LA SAUVEGARDE pour que l'horodatage
                    '                (date de modification) du fichier soit récent.
                    ' =============================================================
                    On Error Resume Next
                    wb.Save
                    
                    If Err.Number = 0 Then
                        OuvrirEtMettreAJourLien = True
                    Else
                        ' En cas d'échec (ex: fichier en lecture seule), on le note
                        OuvrirEtMettreAJourLien = False
                        Err.Clear
                    End If
                    On Error GoTo 0
                Else
                    OuvrirEtMettreAJourLien = False
                    Err.Clear
                End If

                ' Fermer (déjà sauvegardé, on met False pour éviter une double tentative)
                wb.Close SaveChanges:=False
                Exit Function
            End If
        End If
    Next i

    OuvrirEtMettreAJourLien = False
    On Error GoTo 0
End Function



</code></pre>

