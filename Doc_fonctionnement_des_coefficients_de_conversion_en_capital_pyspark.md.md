---


---

<h1 id="chapitre-0.-compréhension-personnelle">Chapitre 0. Compréhension personnelle</h1>
<p>Plusieurs points du traitement des <strong>coefficients de conversion en capital</strong> dans les scripts PySpark prêtent à confusion et méritent d’être clarifiés avant d’entrer dans le détail du code :</p>
<ol>
<li>
<p>Absence de plafonnement à 2018 sur <code>annee_tx</code> (contrairement à <code>annee_s</code> pour la surcote) et rôle de <code>max_annee</code> lors de la duplication</p>
</li>
<li>
<p>Distinction entre la clé de jointure (<code>annee_tx</code>) et le critère de bascule entre ancien et nouveau barème (<code>DDEFFL &gt;= date_new_tarif</code>)</p>
</li>
<li>
<p>Rôle de <code>param_depart_retraite</code> et condition d’âge légal pour les situations <code>01</code> et <code>02</code> (bascule entre rente différée <code>tarif_aff_r</code> et capital <code>tarif_aff_k_new</code>)</p>
</li>
<li>
<p>Fonctionnement des jointures et écrasement temporaire de <code>age_ret_y</code> / <code>age_ret_m</code> dans <code>create_data_pm_tx</code></p>
</li>
</ol>
<h3 id="absence-de-plafonnement-à-2018-sur-annee_tx-et-horizon-de-duplication">1. Absence de plafonnement à 2018 sur <code>annee_tx</code> et horizon de duplication</h3>
<p>À première vue, on pourrait s’attendre à retrouver un plafonnement à <code>2018</code> similaire à celui utilisé pour la surcote (<code>annee_s</code>). Or, pour les tarifs de provisionnement et les coefficients de conversion en capital (<code>create_data_pm_tx</code>), <strong><code>annee_tx</code> n’est pas plafonnée à <code>2018</code></strong>.</p>
<h4 id="pourquoi-annee_tx-nest-elle-pas-plafonnée-à-2018-">Pourquoi <code>annee_tx</code> n’est-elle pas plafonnée à 2018 ?</h4>
<ul>
<li>
<p><strong>Invariance calendaire des barèmes de conversion :</strong> Les fichiers sources (<code>param_conv_kal</code>, <code>param_conv_kal_new</code>, <code>param_conjoint</code>, <code>param_conjoint_new</code>, <code>param_Orphelin</code>, <code>param_Orphelin_new</code>) ne contiennent initialement que l’année <code>2005</code>, mais leurs valeurs ne dépendent que de l’âge (et du barème ancien/nouveau), pas de l’année calendaire.</p>
</li>
<li>
<p><strong>Duplication jusqu’à <code>max_annee</code> :</strong> La fonction <code>apply_param_duplicate_on_table_parametre</code> duplique ces six tables sur toutes les années jusqu’à <code>max_annee = 2035</code> (ou <code>annee_sim + 64</code> dans la variante dynamique commentée).</p>
</li>
<li>
<p><strong>Conséquence sur la jointure :</strong> Lors de la jointure entre <code>data_pm_tx</code> et <code>data_tx_prov</code> sur <code>annee_tx</code>, tant que <code>year(d_sim)</code> (pour les situations <code>01</code>/<code>02</code>) ou <code>year(ddeffl)</code> (pour les situations liquidées <code>03</code>/<code>06</code>/<code>08</code>/<code>09</code>) est inférieur ou égal à <code>max_annee</code> (<code>2035</code>), la ligne correspondante existe directement dans <code>data_tx_prov</code> sans risque de valeur <code>NULL</code>. En revanche, au-delà de <code>2035</code>, si <code>max_annee</code> n’est pas étendu, la jointure produit des valeurs manquantes (<code>pm_with_solde_pm_Null</code>).</p>
</li>
</ul>
<h3 id="distinction-entre-clé-de-jointure-annee_tx-et-choix-du-barème-date_new_tarif">2. Distinction entre clé de jointure (<code>annee_tx</code>) et choix du barème (<code>date_new_tarif</code>)</h3>
<p>Comme pour la surcote, l’année utilisée pour la jointure (<code>annee_tx</code>) <strong>ne sert pas</strong> à choisir entre l’ancien barème (<code>*_k</code>) et le nouveau barème (<code>*_k_new</code>).</p>
<ul>
<li>
<p>Lors de la création de <code>df_tx_conv</code> puis <code>df_tx_prov</code>, <strong>toutes les colonnes tarifaires coexistent simultanément</strong> sur chaque ligne : <code>tarif_aff_k</code>, <code>tarif_aff_k_new</code>, <code>tarif_conj_k</code>, <code>tarif_conj_k_new</code>, <code>tarif_orph_k</code>, <code>tarif_orph_k_new</code>.</p>
</li>
<li>
<p>Lors de la double jointure dans <code>update_data_pm_tx</code>, le programme rattache à chaque individu à la fois les coefficients à l’âge entier <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> et à l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (<code>_p1</code>) pour l’ancien <strong>et</strong> le nouveau barème.</p>
</li>
<li>
<p>C’est uniquement dans la fonction <code>interpol_taux_pm</code> que le programme arbitre entre l’ancien et le nouveau barème, en comparant la date exacte d’effet RAFP <strong><code>DDEFFL</code></strong> au paramètre <strong><code>date_new_tarif</code></strong> (fixé par défaut à <strong><code>"2022-01-01"</code></strong> dans <code>calcul_PM.py</code>) :</p>
</li>
</ul>
<pre><code>Si DDEFFL &gt;= "2022-01-01"  --&gt;  Nouveau barème {tarif_*_k_new ; tarif_*_k_new_p1}
Si DDEFFL &lt;  "2022-01-01"  --&gt;  Ancien barème  {tarif_*_k     ; tarif_*_k_p1}

</code></pre>
<h4 id="conséquence-pour-les-cotisants-01-et-dormants-02">Conséquence pour les cotisants (<code>01</code>) et dormants (<code>02</code>)</h4>
<p>Pour les agents en situation <code>01</code> ou <code>02</code>, la liquidation n’a pas encore eu lieu et <code>DDEFFL</code> est initialisée par convention à <strong><code>2999-12-31</code></strong> (<code>script_importation_PM.py</code>). La condition <code>DDEFFL &gt;= "2022-01-01"</code> étant systématiquement vérifiée (<code>2999-12-31 &gt;= 2022-01-01</code>), ces populations basculent automatiquement sur le <strong>nouveau barème</strong> (<code>tarif_aff_k_new</code>) dès lors qu’elles remplissent les conditions d’âge et de seuil en points.</p>
<blockquote>
<p><strong>Point de vigilance :</strong> Contrairement à <code>update_data_pm_s</code> (qui reçoit <code>date_new_bareme = '2015-03-01'</code> depuis le dictionnaire de paramètres), l’appel <code>update_data_pm_tx(data_pm_tx, df_tx_prov)</code> dans <code>produce_data_pm_traite</code> ne surcharge pas <code>date_new_tarif</code> et utilise donc la valeur par défaut <code>"2022-01-01"</code> codée dans la signature de la fonction.</p>
</blockquote>
<h3 id="rôle-de-param_depart_retraite-et-condition-dâge-légal-pour-les-situations-01-et-02">3. Rôle de <code>param_depart_retraite</code> et condition d’âge légal pour les situations <code>01</code> et <code>02</code></h3>
<p>Pour les cotisants (<code>01</code>) et dormants (<code>02</code>) dont le stock de points est inférieur au seuil de conversion en capital (<code>ctdver == "01"</code>), le coefficient de conversion en capital <strong>n’est pas appliqué tant que l’agent n’a pas atteint son âge légal de départ à la retraite</strong> (<code>Age_ret_y</code>, <code>Age_ret_m</code>) défini dans <code>param_depart_retraite</code>.</p>
<p>Dans <code>interpol_taux_pm</code>, la première condition testée pour <code>situat in ('01', '02')</code> vérifie si l’âge de l’assuré (<code>age_y</code>, <code>age_m</code>) à la date de simulation <code>d_sim</code> est strictement inférieur à son âge légal (<code>age_ret_y</code>, <code>age_ret_m</code>) :</p>
<ul>
<li>
<p><strong>Si l’âge légal n’est pas encore atteint</strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>&lt;</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} &lt; \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> ou (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>=</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} = \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.74056em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_m</mtext><mo>&lt;</mo><mtext>age_ret_m</mtext></mrow><annotation encoding="application/x-tex">\text{age\_m} &lt; \text{age\_ret\_m}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_m</span></span></span></span></span></span>)) : le programme applique le <strong>tarif de rente viagère différée <code>tarif_aff_r</code></strong> (qui intègre l’actualisation et la survie jusqu’à l’âge de la retraite), même si <code>ctdver == "01"</code>.</p>
</li>
<li>
<p><strong>Si l’âge légal est atteint ou dépassé</strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>&gt;</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} &gt; \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&gt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> ou (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>=</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} = \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.74056em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_m</mtext><mo>≥</mo><mtext>age_ret_m</mtext></mrow><annotation encoding="application/x-tex">\text{age\_m} \ge \text{age\_ret\_m}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.94597em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_m</span></span></span></span></span></span>)) et <code>ctdver == "01"</code> : le programme bascule sur le <strong>coefficient de conversion en capital <code>tarif_aff_k_new</code></strong> (ou <code>tarif_aff_k</code> si <code>DDEFFL &lt; 2022-01-01</code>).</p>
</li>
</ul>
<h3 id="fonctionnement-des-jointures-et-écrasement-temporaire-de-age_ret_y--age_ret_m">4. Fonctionnement des jointures et écrasement temporaire de <code>age_ret_y</code> / <code>age_ret_m</code></h3>
<h4 id="structure-des-fichiers-sources">Structure des fichiers sources</h4>
<p>Les six tables d’entrée des coefficients de conversion ne contiennent qu’une seule année (<code>2005</code>) et aucun axe de sexe ni d’âge de départ à la retraite :</p>
<ul>
<li>
<p><strong><code>param_conv_kal.md</code></strong> (<code>conv_Kal</code>) et <strong><code>param_conv_kal_new.md</code></strong> (<code>conv_Kal_new</code>) : renseignées uniquement pour les âges entiers de <strong><code>60</code> à <code>75</code> ans</strong>.</p>
</li>
<li>
<p><strong><code>param_conjoint.md</code></strong> (<code>conjoint</code>) et <strong><code>param_conjoint_new.md</code></strong> (<code>conjoint_new</code>) : renseignées pour les âges entiers de <strong><code>16</code> à <code>103</code> ans</strong>.</p>
</li>
<li>
<p><strong><code>param_Orphelin.md</code></strong> (<code>orphelin_k</code>, <code>orphelin_r</code>) et <strong><code>param_Orphelin_new.md</code></strong> (<code>orphelin_k_new</code>, <code>orphelin_r</code>) : renseignées pour les âges entiers de <strong><code>0</code> à <code>21</code> ans</strong>.</p>
</li>
</ul>
<h4 id="duplication-et-jointure-avec-tx_conv-et-tx_prov">Duplication et jointure avec <code>tx_conv</code> et <code>tx_prov</code></h4>
<p>Contrairement à la surcote (<code>dup_age_ret = False</code>), les six tables de conversion en capital sont dupliquées avec <strong><code>dup_annee = True</code></strong>, <strong><code>dup_sexe = True</code></strong> et <strong><code>dup_age_ret = True</code></strong>. À âge donné, la valeur du coefficient en capital est identique quels que soient le sexe (<code>1</code> ou <code>2</code>) et l’âge de retraite (<code>60</code> à <code>64</code> ans).</p>
<p>Par exemple, dans <code>tx_prov</code> pour <code>annee = 2018</code>, <code>sexe = 1</code> (<code>M</code>) et <code>age = 64</code> :</p>

<table>
<thead>
<tr>
<th>annee</th>
<th>age</th>
<th>ccsex1</th>
<th>Age_ret_y</th>
<th>Age_ret_m</th>
<th>tarif_aff_k</th>
<th>tarif_aff_k_new</th>
<th>tarif_conj_k</th>
<th>tarif_conj_k_new</th>
<th>tarif_orph_k</th>
<th>tarif_orph_k_new</th>
</tr>
</thead>
<tbody>
<tr>
<td>2018</td>
<td>64.0</td>
<td>M</td>
<td>60.0</td>
<td>0</td>
<td>23.22</td>
<td>25.57</td>
<td>22.36</td>
<td>23.48</td>
<td>0.0</td>
<td>0.0</td>
</tr>
<tr>
<td>2018</td>
<td>64.0</td>
<td>M</td>
<td>61.0</td>
<td>0</td>
<td>23.22</td>
<td>25.57</td>
<td>22.36</td>
<td>23.48</td>
<td>0.0</td>
<td>0.0</td>
</tr>
<tr>
<td>2018</td>
<td>64.0</td>
<td>M</td>
<td>62.0</td>
<td>0</td>
<td>23.22</td>
<td>25.57</td>
<td>22.36</td>
<td>23.48</td>
<td>0.0</td>
<td>0.0</td>
</tr>
<tr>
<td>2018</td>
<td>64.0</td>
<td>M</td>
<td>63.0</td>
<td>0</td>
<td>23.22</td>
<td>25.57</td>
<td>22.36</td>
<td>23.48</td>
<td>0.0</td>
<td>0.0</td>
</tr>
<tr>
<td>2018</td>
<td>64.0</td>
<td>M</td>
<td>64.0</td>
<td>0</td>
<td>23.22</td>
<td>25.57</td>
<td>22.36</td>
<td>23.48</td>
<td>0.0</td>
<td>0.0</td>
</tr>
</tbody>
</table><h4 id="jointure-entre-data_tx_prov-et-data_pm_tx">Jointure entre <code>data_tx_prov</code> et <code>data_pm_tx</code></h4>
<p>La jointure s’effectue sur les 5 clés <code>["annee_tx", "mat_tx_y", "ccsex1", "age_ret_y", "age_ret_m"]</code> :</p>
<ol>
<li>
<p><strong>1ère jointure (lecture à l’âge</strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span><strong>)</strong> : <code>data_pm_tx</code> est jointe en <code>left join</code> à <code>data_tx_prov</code> (où <code>annee</code> est renommée en <code>annee_tx</code> et <code>age</code> en <code>mat_tx_y</code>) pour récupérer les tarifs à l’âge entier <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> : <code>tarif_aff_k</code>, <code>tarif_aff_k_new</code>, <code>tarif_conj_k</code>, <code>tarif_conj_k_new</code>, <code>tarif_orph_k</code>, <code>tarif_orph_k_new</code> (ainsi que les tarifs de rente).</p>
</li>
<li>
<p><strong>2nde jointure (lecture à l’âge</strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span><strong>)</strong> : <code>data_tx_prov</code> est renommée avec le suffixe <code>_p1</code> (<code>age_p1</code> devient <code>mat_tx_y_p1</code> et <code>annee_p1</code> devient <code>annee_tx_p1</code>) afin de récupérer les coefficients à l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (<code>tarif_aff_k_p1</code>, <code>tarif_aff_k_new_p1</code>, <code>tarif_conj_k_p1</code>, <code>tarif_conj_k_new_p1</code>, <code>tarif_orph_k_p1</code>, <code>tarif_orph_k_new_p1</code>) nécessaires à l’interpolation linéaire mensuelle.</p>
</li>
</ol>
<p><strong>Astuce technique sur <code>age_ret_y</code> / <code>age_ret_m</code> :</strong> Dans <code>create_data_pm_tx</code>, pour les situations <code>03</code>, <code>05</code>, <code>06</code> et <code>07</code>, le code sauvegarde <code>age_ret_y</code> et <code>age_ret_m</code> dans <code>temp_age_ret_y</code> et <code>temp_age_ret_m</code>, puis force <code>age_ret_m = 0</code> et <code>age_ret_y</code> à <code>60</code> (<code>year(denai) &lt;= 1952</code>), <code>61</code> (<code>1953</code>) ou <code>62</code> (<code>&gt; 1953</code>). Comme les coefficients de conversion en capital sont indépendants de <code>Age_Retraite</code> dans <code>tx_prov</code>, cette normalisation garantit une correspondance exacte sur les 5 clés de jointure (notamment pour les générations 1951–1954 ayant des mois de départ non nuls dans <code>param_depart_retraite</code>, comme <code>60 ans 4 mois</code> ou <code>61 ans 7 mois</code>).</p>
<h1 id="chapitre-1.-pyspark-version-31122025">Chapitre 1. PySpark Version 31/12/2025</h1>
<h2 id="i.-mécanisme-des-coefficients-de-conversion-en-capital-en-pyspark">I. Mécanisme des coefficients de conversion en capital en PySpark</h2>
<p>Le mécanisme de gestion des coefficients de conversion en capital s’articule en cinq étapes successives :</p>
<ol>
<li>
<p><strong>La duplication des barèmes bruts</strong> (<code>param_conv_kal</code>, <code>param_conv_kal_new</code>, <code>param_conjoint</code>, <code>param_conjoint_new</code>, <code>param_Orphelin</code>, <code>param_Orphelin_new</code>) croisée avec la grille des âges légaux (<code>param_depart_retraite</code>) ;</p>
</li>
<li>
<p><strong>Leur consolidation et le bornage des extrémités</strong> dans la table des tarifs de provisionnement (<code>tx_prov</code>) ;</p>
</li>
<li>
<p><strong>La double jointure avec les assurés</strong> aux âges entiers <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> (<code>mat_tx_y</code>) et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (<code>mat_tx_y_p1</code>) ;</p>
</li>
<li>
<p><strong>La sélection du barème et l’interpolation mensuelle</strong> dans <code>taux_pm</code> selon la situation (<code>situat</code>), le mode de versement (<code>ctdver</code>) et la date d’effet (<code>DDEFFL</code>) ;</p>
</li>
<li>
<p><strong>La valorisation de la Provision Mathématique (PM)</strong> après sélection du millésime de valeur de service (<code>vs</code>) et déduction éventuelle des prestations versées.</p>
</li>
</ol>
<p>Lors des jointures entre la table des assurés et la table des tarifs, <strong><code>annee_tx</code></strong> correspond à l’année calendaire du barème (<code>annee</code>) tandis que <strong><code>mat_tx_y</code></strong> correspond à l’âge de maturité en années entières (<code>age</code>).</p>
<h3 id="duplication-des-barèmes-de-conversion-en-capital-apply_param_duplicate_on_table_parametre">1. Duplication des barèmes de conversion en capital (<code>apply_param_duplicate_on_table_parametre</code>)</h3>
<p>Les barèmes initiaux sont chargés depuis six tables distinctes couvrant les trois catégories de bénéficiaires (affiliés, conjoints, orphelins) en ancien et nouveau barème :</p>
<ul>
<li>
<p><strong>Structure brute :</strong> Les fichiers sources ne contiennent qu’une seule année de référence (<code>annee = 2005</code>), sans distinction de sexe ni d’âge légal de départ à la retraite, sur des plages d’âges propres à chaque population (<code>60–75</code> ans pour l’affilié, <code>16–103</code> ans pour le conjoint, <code>0–21</code> ans pour l’orphelin).</p>
</li>
<li>
<p><strong>Duplication multi-axes :</strong> La fonction <code>param_duplicate</code> projette chacune de ces six tables sur les années (<code>dup_annee = True</code>, jusqu’à <code>max_annee = 2035</code>), sur les deux sexes (<code>dup_sexe = True</code>, valeurs <code>1</code> et <code>2</code>), et sur les âges entiers de départ à la retraite (<code>dup_age_ret = True</code>, de <code>age_retraite_annee_min</code> à <code>age_retraite_annee_max</code> issus de <code>param_depart_retraite</code>).</p>
</li>
<li>
<p><strong>Renommage spécifique orphelin :</strong> Dans <code>param_orphelin_new</code>, la colonne <code>orphelin_r</code> est renommée en <code>orphelin_r_new</code> afin d’éviter toute collision de colonnes lors des jointures ultérieures.</p>
</li>
</ul>
<h3 id="consolidation-et-remplissage-des-bornes-create_tx_conv-et-create_tx_prov">2. Consolidation et remplissage des bornes (<code>create_tx_conv</code> et <code>create_tx_prov</code>)</h3>
<p>Une fois dupliquées, les six tables de conversion en capital sont intégrées à la table générale des taux de provisionnement <code>tx_prov</code> :</p>
<ul>
<li>
<p><strong>Grille commune (<code>create_tx_conv</code>) :</strong> Toutes les tables de paramètres sont unifiées par jointure externe (<code>outer join</code>) sur les 4 clés <code>["annee", "age", "sexe", "Age_Retraite"]</code>, puis chacune des tables de conversion est rattachée par jointure gauche (<code>left join</code>).</p>
</li>
<li>
<p><strong>Mapping des tarifs en capital (<code>create_tx_prov</code>) :</strong> Après jointure avec les tarifs de rentes (<code>df_tx_rente</code>), les colonnes brutes sont copiées dans des variables standardisées : <code>tarif_aff_k</code> (<code>conv_Kal</code>), <code>tarif_aff_k_new</code> (<code>conv_Kal_new</code>), <code>tarif_conj_k</code> (<code>conjoint</code>), <code>tarif_conj_k_new</code> (<code>conjoint_new</code>), <code>tarif_orph_k</code> (<code>orphelin_k</code>) et <code>tarif_orph_k_new</code> (<code>orphelin_k_new</code>).</p>
</li>
<li>
<p><strong>Imputation des âges hors barème :</strong> La fonction <code>create_summary_min_max_from_spark_dataframe</code> calcule les bornes <code>_min</code> et <code>_max</code> par groupe <code>["annee", "Age_Retraite"]</code>. Pour les affiliés et conjoints, les valeurs <code>NULL</code> avant l’âge de retraite (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span>) sont remplacées par le <strong>plafond <code>_max</code></strong> (valeur au premier âge du barème), et celles à partir de l’âge de retraite (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>≥</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} \ge \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.83041em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span>) par le <strong>plancher <code>_min</code></strong> (valeur au dernier âge du barème). Pour les orphelins, toute valeur <code>NULL</code> (au-delà de 21 ans) est forcée à <strong><code>_min</code> (<code>0.0</code>)</strong>.</p>
</li>
<li>
<p><strong>Préparation des clés de jointure (<code>update_tx_prov</code>) :</strong> La table est dupliquée sur les couples <code>(Age_ret_y, Age_ret_m)</code> issus de <code>param_depart_retraite</code> complétés par les lignes <code>Age_ret_m = 0</code> (<code>fill_param_retraite_with_age_ret_m_null</code>). Contrairement à <code>tarif_aff_r</code> qui est interpolé selon <code>Age_ret_m</code>, les tarifs en capital (<code>tarif_*_k</code> et <code>tarif_*_k_new</code>) sont conservés tels quels sur chaque couple <code>(Age_ret_y, Age_ret_m)</code>.</p>
</li>
</ul>
<h3 id="rôle-de-annee_tx-et-mat_tx_y-et-double-jointure-update_data_pm_tx">3. Rôle de <code>annee_tx</code> et <code>mat_tx_y</code> et double jointure (<code>update_data_pm_tx</code>)</h3>
<p>À l’étape <code>PM_tx</code> (<code>create_data_pm_tx</code>), le programme calcule pour chaque contrat le millésime <code>annee_tx</code> et la maturité en années entières (<code>mat_tx_y</code>, <code>mat_tx_y_p1 = mat_tx_y + 1</code>) et mois résiduels (<code>mat_tx_m</code>) :</p>
<ul>
<li>
<p><strong>Correspondance des clés :</strong> Dans <code>data_tx_prov</code>, <code>annee</code> est renommée en <code>annee_tx</code> et <code>age</code> en <code>mat_tx_y</code>.</p>
</li>
<li>
<p><strong>Double jointure sur les 5 clés <code>["annee_tx", "mat_tx_y", "ccsex1", "age_ret_y", "age_ret_m"]</code> :</strong> Une première jointure gauche récupère les tarifs à l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span>, et une seconde jointure gauche sur <code>(annee_tx_p1, mat_tx_y_p1)</code> récupère les tarifs à l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (suffixés <code>_p1</code>).</p>
</li>
<li>
<p><strong>Passif de l’exercice précédent (</strong><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N-1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span><strong>) :</strong> Le même mécanisme est exécuté dans <code>create_data_pm_tx1</code> et <code>update_data_pm_tx1</code> à partir de <code>situn1</code> et <code>d_sim1</code> pour alimenter <code>taux_pm1</code>.</p>
</li>
</ul>
<h3 id="choix-de-la-colonne-et-critères-dinterpolation-interpol_taux_pm">4. Choix de la colonne et critères d’interpolation (<code>interpol_taux_pm</code>)</h3>
<p>Une fois les colonnes tarifaires à <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> rattachées, la fonction <code>interpol_taux_pm</code> détermine la valeur finale de <strong><code>taux_pm</code></strong> en croisant :</p>
<ul>
<li>
<p><strong>Le code situation (<code>situat</code>)</strong> : droit direct (<code>01</code>, <code>02</code>, <code>03</code>, <code>06</code>), réversion conjoint (<code>08</code>), réversion orphelin (<code>09</code>), ou cas sans droit direct/réversataire (<code>04</code>, <code>05</code>, <code>07</code>, <code>X</code>, <code>Y</code>) ;</p>
</li>
<li>
<p><strong>Le mode de versement (<code>ctdver</code>)</strong> : <code>"01"</code> pour un versement en <strong>Capital</strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>s_pts</mtext><mo>&lt;</mo><mtext>seuil_K_vs_R</mtext></mrow><annotation encoding="application/x-tex">\text{s\_pts} &lt; \text{seuil\_K\_vs\_R}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">seuil_K_vs_R</span></span></span></span></span></span>), <code>"02"</code> pour un versement en <strong>Rente</strong> ;</p>
</li>
<li>
<p><strong>La condition d’âge légal</strong> (pour <code>01</code>, <code>02</code> et <code>06</code> en capital) : bascule sur <code>tarif_aff_r</code> tant que <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo stretchy="false">(</mo><mtext>age_y</mtext><mo separator="true">,</mo><mtext>age_m</mtext><mo stretchy="false">)</mo><mo>&lt;</mo><mo stretchy="false">(</mo><mtext>age_ret_y</mtext><mo separator="true">,</mo><mtext>age_ret_m</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">(\text{age\_y}, \text{age\_m}) &lt; (\text{age\_ret\_y}, \text{age\_ret\_m})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mopen">(</span><span class="mord text"><span class="mord">age_y</span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord text"><span class="mord">age_m</span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mopen">(</span><span class="mord text"><span class="mord">age_ret_y</span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord text"><span class="mord">age_ret_m</span></span><span class="mclose">)</span></span></span></span></span> ;</p>
</li>
<li>
<p><strong>La date d’effet RAFP (<code>DDEFFL</code>)</strong> : bascule sur le nouveau barème <code>*_k_new</code> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>date_new_tarif</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{date\_new\_tarif}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">date_new_tarif</span></span></span></span></span></span> (<code>"2022-01-01"</code>), sinon maintien de l’ancien barème <code>*_k</code>.</p>
</li>
</ul>
<p>Le coefficient sélectionné est ensuite interpolé linéairement entre l’âge entier <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> et l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> au prorata des mois résiduels <code>mat_tx_m</code> :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>taux_pm</mtext><mo>=</mo><mtext>tarif</mtext><mo>×</mo><mfrac><mrow><mn>12</mn><mo>−</mo><mtext>mat_tx_m</mtext></mrow><mn>12</mn></mfrac><mo>+</mo><mtext>tarif_p1</mtext><mo>×</mo><mfrac><mtext>mat_tx_m</mtext><mn>12</mn></mfrac></mrow><annotation encoding="application/x-tex">
\text{taux\_pm} = \text{tarif} \times \frac{12 - \text{mat\_tx\_m}}{12} + \text{tarif\_p1} \times \frac{\text{mat\_tx\_m}}{12}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.77777em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">tarif</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.03044em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.34444em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.7em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord text"><span class="mord">mat_tx_m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">tarif_p1</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.00108em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.31508em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.7em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord text"><span class="mord">mat_tx_m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></p>
<h3 id="application-au-calcul-des-provisions-mathématiques-create_data_pm_p">5. Application au calcul des Provisions Mathématiques (<code>create_data_pm_p</code>)</h3>
<p>Lorsque <code>ctdver == "01"</code>, <code>taux_pm</code> porte le coefficient de conversion en capital interpolé et intervient directement dans la valorisation des six colonnes de provisions mathématiques (à valeur de service projetée <code>vs</code> ou constante <code>vs_cst</code>), après déduction des prestations déjà versées (<code>prestation</code>, <code>prestation_1</code>, <code>prestation_exo</code>) :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>pm_solde</mtext><mo>=</mo><mtext>s_pts</mtext><mo>×</mo><mtext>vs</mtext><mo>×</mo><mtext>taux_pm</mtext><mo>×</mo><mtext>surcote</mtext><mo>−</mo><mtext>prestation</mtext></mrow><annotation encoding="application/x-tex">
\text{pm\_solde} = \text{s\_pts} \times \text{vs} \times \text{taux\_pm} \times \text{surcote} - \text{prestation}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">pm_solde</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.66666em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">vs</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.69841em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">surcote</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.8623em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">prestation</span></span></span></span></span></span></span></p>
<h2 id="ii.-duplication-des-barèmes-de-conversion-en-capital-et-rôle-de-param_depart_retraite">II. Duplication des barèmes de conversion en capital et rôle de <code>param_depart_retraite</code></h2>
<p>La fonction <code>apply_param_duplicate_on_table_parametre</code> (définie dans <code>creation_table_prov.py</code> et <code>importation_table_PM_source.py</code>) prépare les six tables de conversion en capital avant leur export au format Parquet sur HDFS.</p>
<h3 id="caractéristiques-des-six-tables-dentrée">1. Caractéristiques des six tables d’entrée</h3>

<table>
<thead>
<tr>
<th>Table source</th>
<th>Bénéficiaire</th>
<th>Régime tarifaire</th>
<th>Colonne(s) valeur(s)</th>
<th>Plage d’âges brute</th>
<th>Valeur min (âge max)</th>
<th>Valeur max (âge min)</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>param_conv_kal.md</code></td>
<td>Affilié</td>
<td>Ancien barème</td>
<td><code>conv_Kal</code></td>
<td><code>60</code> à <code>75</code> ans</td>
<td><code>15.24</code> (75 ans)</td>
<td><code>25.98</code> (60 ans)</td>
</tr>
<tr>
<td><code>param_conv_kal_new.md</code></td>
<td>Affilié</td>
<td>Nouveau barème</td>
<td><code>conv_Kal_new</code></td>
<td><code>60</code> à <code>75</code> ans</td>
<td><code>17.07</code> (75 ans)</td>
<td><code>27.11</code> (60–62 ans)</td>
</tr>
<tr>
<td><code>param_conjoint.md</code></td>
<td>Conjoint</td>
<td>Ancien barème</td>
<td><code>conjoint</code></td>
<td><code>16</code> à <code>103</code> ans</td>
<td><code>1.93</code> (103 ans)</td>
<td><code>41.98</code> (16–31 ans)</td>
</tr>
<tr>
<td><code>param_conjoint_new.md</code></td>
<td>Conjoint</td>
<td>Nouveau barème</td>
<td><code>conjoint_new</code></td>
<td><code>16</code> à <code>103</code> ans</td>
<td><code>2.25</code> (103 ans)</td>
<td><code>46.71</code> (16–31 ans)</td>
</tr>
<tr>
<td><code>param_Orphelin.md</code></td>
<td>Orphelin</td>
<td>Ancien barème</td>
<td><code>orphelin_k</code>, <code>orphelin_r</code></td>
<td><code>0</code> à <code>21</code> ans</td>
<td><code>0.00</code> (21 ans)</td>
<td><code>18.33</code> / <code>18.96</code> (0 an)</td>
</tr>
<tr>
<td><code>param_Orphelin_new.md</code></td>
<td>Orphelin</td>
<td>Nouveau barème</td>
<td><code>orphelin_k_new</code>, <code>orphelin_r</code> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>→</mo></mrow><annotation encoding="application/x-tex">\rightarrow</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">→</span></span></span></span></span> <code>orphelin_r_new</code>)</td>
<td><code>0</code> à <code>21</code> ans</td>
<td><code>0.00</code> (21 ans)</td>
<td><code>18.96</code> (0 an)</td>
</tr>
</tbody>
</table><h3 id="rôle-de-param_retratie.md-param_depart_retraite">2. Rôle de <code>param_retratie.md</code> (<code>param_depart_retraite</code>)</h3>
<p>La table <code>param_depart_retraite</code> (<code>param_retratie.md</code>) structure le calendrier légal de départ à la retraite par génération de naissance (<code>Deb_naissance</code> à <code>Fin_naissance</code>) :</p>

<table>
<thead>
<tr>
<th>Deb_naissance</th>
<th>Fin_naissance</th>
<th>Age_ret_y</th>
<th>Age_ret_m</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>02/01/1900</code></td>
<td><code>30/06/1951</code></td>
<td><code>60</code></td>
<td><code>0</code></td>
</tr>
<tr>
<td><code>01/07/1951</code></td>
<td><code>31/12/1951</code></td>
<td><code>60</code></td>
<td><code>4</code></td>
</tr>
<tr>
<td><code>01/01/1952</code></td>
<td><code>31/12/1952</code></td>
<td><code>60</code></td>
<td><code>9</code></td>
</tr>
<tr>
<td><code>01/01/1953</code></td>
<td><code>31/12/1953</code></td>
<td><code>61</code></td>
<td><code>2</code></td>
</tr>
<tr>
<td><code>01/01/1954</code></td>
<td><code>31/12/1954</code></td>
<td><code>61</code></td>
<td><code>7</code></td>
</tr>
<tr>
<td><code>01/01/1955</code></td>
<td><code>31/12/1955</code></td>
<td><code>62</code></td>
<td><code>0</code></td>
</tr>
<tr>
<td><code>01/01/1956</code></td>
<td><code>31/12/2999</code></td>
<td><code>62</code></td>
<td><code>0</code></td>
</tr>
</tbody>
</table><p>Cette table intervient à trois niveaux dans la chaîne de calcul des coefficients en capital :</p>
<ol>
<li>
<p><strong>Enrichissement individuel (<code>update_data_PM_with_add_age_depart_retraite</code>) :</strong> Chaque assuré reçoit son âge légal (<code>Age_ret_y</code>, <code>Age_ret_m</code>) par jointure d’intervalle entre sa date de naissance <code>DENAI</code> et <code>[Deb_naissance, Fin_naissance]</code>.</p>
</li>
<li>
<p><strong>Duplication sur l’axe <code>Age_Retraite</code> (<code>param_duplicate</code>) :</strong> <code>param_duplicate</code> extrait les bornes <code>age_retraite_annee_min</code> et <code>age_retraite_annee_max</code> de <code>param_depart_retraite</code> et duplique les six barèmes de conversion en capital sur chaque âge entier de retraite (<code>dup_age_ret = True</code>), en parallèle de la duplication sur <code>annee</code> (<code>2005</code> à <code>2035</code>) et <code>sexe</code> (<code>1</code> et <code>2</code>).</p>
</li>
<li>
<p><strong>Complétude des couples <code>(Age_ret_y, Age_ret_m)</code> (<code>fill_param_retraite_with_age_ret_m_null</code>) :</strong> Dans <code>update_tx_prov</code>, la fonction <code>fill_param_retraite_with_age_ret_m_null</code> s’assure que chaque année entière <code>Age_ret_y</code> (comme <code>60</code> ou <code>61</code>) possède également une entrée avec <code>Age_ret_m = 0</code>. Cela garantit que l’écrasement temporaire <code>age_ret_m = 0</code> appliqué aux situations <code>03</code>/<code>05</code>/<code>06</code>/<code>07</code> dans <code>create_data_pm_tx</code> trouve toujours une ligne correspondante dans <code>data_tx_prov</code>.</p>
</li>
</ol>
<h2 id="iii.-consolidation-des-coefficients-de-conversion-dans-tx_prov">III. Consolidation des coefficients de conversion dans <code>tx_prov</code></h2>
<p>La construction de la table consolidée <code>tx_prov</code> s’opère en trois étapes dans <code>creation_table_prov.py</code> :</p>
<ol>
<li>
<p><strong>Constitution du référentiel des clés (<code>create_tx_conv</code>) :</strong><br>
Les 8 tables de paramètres dupliquées (<code>param_conv_kal</code>, <code>param_conv_kal_new</code>, <code>param_conjoint</code>, <code>param_conjoint_new</code>, <code>param_orphelin</code>, <code>param_orphelin_new</code>, <code>param_surcote</code>, <code>param_surcotenv</code>) sont assemblées par <code>outer join</code> sur <code>["annee", "age", "sexe", "Age_Retraite"]</code> pour former le squelette <code>df_param_key</code>, puis jointes en <code>left join</code> afin de rassembler toutes les colonnes de coefficients sur chaque combinaison <code>(annee, age, sexe, Age_Retraite)</code>.</p>
</li>
<li>
<p><strong>Fusion avec les primes de rente et bornage des valeurs nulles (<code>create_tx_prov</code>) :</strong><br>
Après jointure gauche avec <code>df_tx_rente</code> (primes pures calculées à partir des tables de mortalité <code>tableh</code> et <code>tablef</code>), les colonnes de conversion en capital sont renommées (<code>tarif_aff_k</code>, <code>tarif_aff_k_new</code>, <code>tarif_conj_k</code>, <code>tarif_conj_k_new</code>, <code>tarif_orph_k</code>, <code>tarif_orph_k_new</code>).<br>
Comme les âges couverts par <code>tx_prov</code> s’étendent de <code>0</code> à <code>103</code> ans, les plages non renseignées dans les barèmes sources (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mn>60</mn></mrow><annotation encoding="application/x-tex">\text{age} &lt; 60</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">60</span></span></span></span></span> ou <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&gt;</mo><mn>75</mn></mrow><annotation encoding="application/x-tex">\text{age} &gt; 75</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&gt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">75</span></span></span></span></span> pour l’affilié ; <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mn>16</mn></mrow><annotation encoding="application/x-tex">\text{age} &lt; 16</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">16</span></span></span></span></span> pour le conjoint ; <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&gt;</mo><mn>21</mn></mrow><annotation encoding="application/x-tex">\text{age} &gt; 21</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&gt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">21</span></span></span></span></span> pour l’orphelin) sont comblées à partir des extremums calculés par groupe <code>["annee", "Age_Retraite"]</code> :</p>
</li>
</ol>

<table>
<thead>
<tr>
<th>Colonnes tarifaires concernées</th>
<th>Condition sur <code>age</code></th>
<th>Règle d’imputation des <code>NULL</code></th>
<th>Valeur imputée (exemple 2018)</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>tarif_aff_k</code>, <code>tarif_aff_k_new</code></td>
<td><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span></td>
<td>Remplacement par <code>*_max</code></td>
<td><code>25.98</code> / <code>27.11</code> (plafond à 60 ans)</td>
</tr>
<tr>
<td><code>tarif_aff_k</code>, <code>tarif_aff_k_new</code></td>
<td><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>≥</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} \ge \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.83041em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span></td>
<td>Remplacement par <code>*_min</code></td>
<td><code>15.24</code> / <code>17.07</code> (plancher à 75 ans)</td>
</tr>
<tr>
<td><code>tarif_conj_k</code>, <code>tarif_conj_k_new</code></td>
<td><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span></td>
<td>Remplacement par <code>*_max</code></td>
<td><code>41.98</code> / <code>46.71</code> (plafond à 16–31 ans)</td>
</tr>
<tr>
<td><code>tarif_conj_k</code>, <code>tarif_conj_k_new</code></td>
<td><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>≥</mo><mtext>Age_Retraite</mtext></mrow><annotation encoding="application/x-tex">\text{age} \ge \text{Age\_Retraite}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.83041em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.99333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_Retraite</span></span></span></span></span></span></td>
<td>Remplacement par <code>*_min</code></td>
<td><code>1.93</code> / <code>2.25</code> (plancher à 103 ans)</td>
</tr>
<tr>
<td><code>tarif_orph_k</code>, <code>tarif_orph_k_new</code>, <code>tarif_orph_r</code></td>
<td>Tout <code>age</code> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>&gt;</mo><mn>21</mn></mrow><annotation encoding="application/x-tex">&gt; 21</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.5782em; vertical-align: -0.0391em;"></span><span class="mrel">&gt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">21</span></span></span></span></span> ans)</td>
<td>Remplacement par <code>*_min</code></td>
<td><code>0.00</code> (extinction des droits à 21 ans)</td>
</tr>
</tbody>
</table><ol start="3">
<li><strong>Déclinaison par mois d’âge légal (<code>update_tx_prov</code>) :</strong><br>
La table est jointe à <code>param_depart_retraite_keys</code> sur <code>Age_ret_y</code> pour démultiplier les lignes selon les mois de départ <code>Age_ret_m</code>. Seul <code>tarif_aff_r</code> (rente viagère différée) fait l’objet d’une interpolation entre <code>Age_ret_y</code> et <code>Age_ret_y - 1</code> au prorata de <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>Age_ret_m</mtext><mi mathvariant="normal">/</mi><mn>12</mn></mrow><annotation encoding="application/x-tex">\text{Age\_ret\_m} / 12</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">Age_ret_m</span></span><span class="mord">/12</span></span></span></span></span> ; les six colonnes de conversion en capital (<code>tarif_*_k</code> et <code>tarif_*_k_new</code>) conservent leur valeur exacte par âge entier <code>age</code>.</li>
</ol>
<h2 id="iv.-rôle-de-annee_tx-et-mat_tx_y-et-double-jointure-par-situation-situat">IV. Rôle de <code>annee_tx</code> et <code>mat_tx_y</code> et double jointure par situation (<code>situat</code>)</h2>
<p>Les variables <code>annee_tx</code>, <code>mat_tx_y</code> et <code>mat_tx_m</code> sont calculées dans <code>create_data_pm_tx</code> (<code>calcul_PM.py</code>) avant d’être utilisées dans la double jointure <code>update_data_pm_tx</code>. Leur définition dépend de la situation de l’individu (<code>situat</code>) et du type de versement (<code>ctdver</code> : <code>'01'</code> = Capital, <code>'02'</code> = Rente).</p>
<h3 id="cotisants-et-dormants-situat-in-01-02">1. Cotisants et Dormants (<code>situat in ('01', '02')</code>)</h3>
<p>Pour ces populations actives ou dormantes, l’indicateur de maturité est paramétré à <code>indice_mat_tx_aff = 3</code> (<code>method = 3</code>, <code>date_deb = denai</code>, <code>date_fin = d_sim</code>) :</p>
<ul>
<li>
<p><strong><code>annee_tx</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>d_sim</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">= \text{year}(\text{d\_sim})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">d_sim</span></span><span class="mclose">)</span></span></span></span></span> (non plafonnée à <code>2018</code>) ;</p>
</li>
<li>
<p><strong><code>mat_tx_y</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>d_sim</mtext><mo stretchy="false">)</mo><mo>−</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>denai</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">= \text{year}(\text{d\_sim}) - \text{year}(\text{denai})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">d_sim</span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">denai</span></span><span class="mclose">)</span></span></span></span></span> (âge atteint par différence de millésimes, issu de la méthode 3) ;</p>
</li>
<li>
<p><strong><code>mat_tx_m</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mn>0</mn></mrow><annotation encoding="application/x-tex">= 0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span> (issu de la méthode 3) ;</p>
</li>
<li>
<p><strong><code>age_ret_y</code> / <code>age_ret_m</code></strong> : âge légal issu de <code>param_depart_retraite</code> selon <code>denai</code>.</p>
</li>
</ul>
<p><strong>Conséquence sur l’interpolation et le choix du tarif :</strong><br>
Comme <code>mat_tx_m = 0</code>, le poids sur l’âge <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (<code>*_p1</code>) est nul et le coefficient est lu directement à l’âge entier <code>mat_tx_y</code> :</p>
<ul>
<li>
<p><strong>Si l’âge exact à <code>d_sim</code> est inférieur à l’âge légal</strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>&lt;</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} &lt; \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> ou (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>=</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} = \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.74056em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_m</mtext><mo>&lt;</mo><mtext>age_ret_m</mtext></mrow><annotation encoding="application/x-tex">\text{age\_m} &lt; \text{age\_ret\_m}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_m</span></span></span></span></span></span>)) : application de <strong><code>tarif_aff_r</code></strong> (rente différée), même lorsque <code>ctdver == '01'</code>.</p>
</li>
<li>
<p><strong>Si l’âge légal est atteint ou dépassé et <code>ctdver == '01'</code></strong> : application de <strong><code>tarif_aff_k_new</code></strong> (car <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>=</mo><mtext>2999-12-31</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} = \text{2999-12-31} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.78041em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">2999-12-31</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>), lu à l’âge <code>mat_tx_y</code> (et plafonné automatiquement à <code>27.11</code> entre 60 et 62 ans, puis borné à <code>17.07</code> au-delà de 75 ans).</p>
</li>
<li>
<p><strong>Si <code>ctdver == '02'</code></strong> : application de <strong><code>tarif_aff_r</code></strong>.</p>
</li>
</ul>
<h3 id="pensionnés-situat--03">2. Pensionnés (<code>situat == '03'</code>)</h3>
<p>Pour les assurés ayant déjà liquidé leurs droits à la date d’effet RAFP <code>ddeffl</code>, les règles se scindent selon <code>ctdver</code> :</p>
<ul>
<li>
<p><strong>Cas liquidation en Capital (<code>ctdver == '01'</code>) :</strong></p>
<ul>
<li>
<p><code>method = 1</code>, <code>date_deb = denai</code>, <code>date_fin = ddeffl</code> ;</p>
</li>
<li>
<p><strong><code>annee_tx</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>ddeffl</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">= \text{year}(\text{ddeffl})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">ddeffl</span></span><span class="mclose">)</span></span></span></span></span> ;</p>
</li>
<li>
<p><strong><code>mat_tx_y</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mo stretchy="false">⌊</mo><mtext>months_between</mtext><mo stretchy="false">(</mo><mtext>ddeffl</mtext><mo separator="true">,</mo><mtext>denai</mtext><mo stretchy="false">)</mo><mi mathvariant="normal">/</mi><mn>12</mn><mo stretchy="false">⌋</mo></mrow><annotation encoding="application/x-tex">= \lfloor \text{months\_between}(\text{ddeffl}, \text{denai}) / 12 \rfloor</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mopen">⌊</span><span class="mord text"><span class="mord">months_between</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">ddeffl</span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord text"><span class="mord">denai</span></span><span class="mclose">)</span><span class="mord">/12</span><span class="mclose">⌋</span></span></span></span></span> (âge révolu en années entières à la date de liquidation) ;</p>
</li>
<li>
<p><strong><code>mat_tx_m</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mo stretchy="false">⌊</mo><mtext>months_between</mtext><mo stretchy="false">(</mo><mtext>ddeffl</mtext><mo separator="true">,</mo><mtext>denai</mtext><mo stretchy="false">)</mo><mo stretchy="false">⌋</mo><mspace></mspace><mspace width="0.4444444444444444em"></mspace><mo stretchy="false">(</mo><mrow><mi mathvariant="normal">m</mi><mi mathvariant="normal">o</mi><mi mathvariant="normal">d</mi></mrow><mspace width="0.3333333333333333em"></mspace><mn>12</mn><mo stretchy="false">)</mo><mo>∈</mo><mo stretchy="false">{</mo><mn>0</mn><mo separator="true">,</mo><mo>…</mo><mo separator="true">,</mo><mn>11</mn><mo stretchy="false">}</mo></mrow><annotation encoding="application/x-tex">= \lfloor \text{months\_between}(\text{ddeffl}, \text{denai}) \rfloor \pmod{12} \in \{0, \dots, 11\}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mopen">⌊</span><span class="mord text"><span class="mord">months_between</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">ddeffl</span></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord text"><span class="mord">denai</span></span><span class="mclose">)⌋</span><span class="mspace allowbreak"></span><span class="mspace" style="margin-right: 0.444444em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mopen">(</span><span class="mord"><span class="mord"><span class="mord mathrm">mod</span></span></span><span class="mspace" style="margin-right: 0.333333em;"></span><span class="mord">12</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">∈</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mopen">{</span><span class="mord">0</span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="minner">…</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mpunct">,</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">11</span><span class="mclose">}</span></span></span></span></span> (mois complets résiduels à la liquidation) ;</p>
</li>
<li>
<p><strong><code>age_ret_y</code> / <code>age_ret_m</code></strong> : temporairement forcés à <code>60</code>/<code>61</code>/<code>62</code> ans et <code>0</code> mois pour garantir l’appariement dans <code>data_tx_prov</code>.</p>
</li>
<li>
<p><strong>Conséquence sur l’interpolation :</strong> Le coefficient de conversion en capital est interpolé linéairement entre l’âge à la liquidation <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>A</mi><mo>=</mo><mtext>mat_tx_y</mtext></mrow><annotation encoding="application/x-tex">A = \text{mat\_tx\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal">A</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_y</span></span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>A</mi><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">A+1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal">A</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> sur <strong><code>tarif_aff_k_new</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>) ou sur <strong><code>tarif_aff_k</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>&lt;</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} &lt; \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.72243em; vertical-align: -0.0391em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>).</p>
</li>
</ul>
</li>
<li>
<p><strong>Cas liquidation en Rente (<code>ctdver == '02'</code>) :</strong></p>
<ul>
<li>
<p><code>method = 4</code>, <code>date_deb = denai</code>, <code>date_fin = d_sim</code>, <code>annee_tx = year(d_sim)</code> ;</p>
</li>
<li>
<p>Application de <strong><code>tarif_aff_r</code></strong> (rente viagère immédiate).</p>
</li>
</ul>
</li>
</ul>
<h3 id="affiliés-décédés-avant-et-après-effet-rafp-situat-in-04-05-06-07">3. Affiliés décédés avant et après effet RAFP (<code>situat in ('04', '05', '06', '07')</code>)</h3>
<ul>
<li>
<p><strong>Décédés avec réversataires (<code>situat in ('05', '07')</code>) :</strong> Aucun tarif n’est joint (<code>annee_tx = NULL</code>, <code>method = NULL</code>) et <code>taux_pm</code> est forcé à <strong><code>0</code></strong> (<code>pm_solde = 0</code>), les engagements étant portés directement sur les lignes des réversataires (<code>08</code> et <code>09</code>).</p>
</li>
<li>
<p><strong>Décédés avant effet RAFP sans réversataire (<code>situat == '04'</code>) :</strong></p>
<ul>
<li>
<p><code>method = 1</code>, <code>date_deb = denai</code>, <code>date_fin = ddesh</code>, <code>annee_tx = year(ddesh)</code> ;</p>
</li>
<li>
<p>Quel que soit <code>ctdver</code>, <code>interpol_taux_pm</code> applique le tarif de décès <strong><code>tarif_dcd</code></strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>rev_imm</mtext><mo>+</mo><mtext>orph_imm</mtext></mrow><annotation encoding="application/x-tex">\text{rev\_imm} + \text{orph\_imm}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">rev_imm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">orph_imm</span></span></span></span></span></span>).</p>
</li>
</ul>
</li>
<li>
<p><strong>Décédés après effet RAFP sans réversataire (<code>situat == '06'</code>) :</strong></p>
<ul>
<li>
<p><strong>Si <code>ctdver == '01'</code> (Capital) :</strong> <code>method = 1</code>, <code>date_deb = denai</code>, <code>date_fin = ddeffl</code>, <code>annee_tx = year(ddeffl)</code>. Dans <code>interpol_taux_pm</code>, cette sous-population est traitée dans la première branche (<code>(situat == '06') &amp; (ctdver == '01')</code>) : elle reçoit <code>tarif_aff_r</code> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>age_ret</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{age\_ret}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret</span></span></span></span></span></span>, sinon <strong><code>tarif_aff_k_new</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>) ou <strong><code>tarif_aff_k</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>&lt;</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} &lt; \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.72243em; vertical-align: -0.0391em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>).</p>
</li>
<li>
<p><strong>Si <code>ctdver == '02'</code> (Rente) :</strong> <code>method = 1</code>, <code>date_deb = denai</code>, <code>date_fin = ddesh</code>, <code>annee_tx = year(ddesh)</code> (ajusté à <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>annee_tx</mtext><mo>−</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">\text{annee\_tx} - 1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">annee_tx</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> si l’anniversaire de naissance est postérieur à <code>ddesh</code> dans l’année), et application de <strong><code>tarif_dcd</code></strong>.</p>
</li>
</ul>
</li>
</ul>
<h3 id="conjoints-et-orphelins-réversataires-situat-in-08-09">4. Conjoints et Orphelins réversataires (<code>situat in ('08', '09')</code>)</h3>
<p>Pour les réversataires (<code>bcraff != bcrpns</code>), les points ont préalablement été proratisés par le taux de réversion (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>tkrev1</mtext><mi mathvariant="normal">/</mi><mn>100</mn></mrow><annotation encoding="application/x-tex">\text{tkrev1} / 100</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord text"><span class="mord">tkrev1</span></span><span class="mord">/100</span></span></span></span></span>) dans <code>calcul_impact_reversion</code> et le code <code>ctdver</code> a été déterminé sur ces points réversibles. Le calcul de <code>taux_pm</code> utilise la date de naissance <strong><code>denai</code> du réversataire</strong> (conjoint ou enfant) :</p>
<ul>
<li>
<p><strong>Cas réversion versée en Capital (<code>ctdver == '01'</code>) :</strong></p>
<ul>
<li>
<p><code>method = 1</code>, <code>date_deb = denai</code> (naissance du réversataire), <code>date_fin = ddeffl</code> ;</p>
</li>
<li>
<p><strong><code>annee_tx</code></strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>ddeffl</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">= \text{year}(\text{ddeffl})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.36687em; vertical-align: 0em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">ddeffl</span></span><span class="mclose">)</span></span></span></span></span> ;</p>
</li>
<li>
<p><strong><code>mat_tx_y</code></strong> et <strong><code>mat_tx_m</code></strong> : âge exact du conjoint (<code>08</code>) ou de l’orphelin (<code>09</code>) à la date d’effet <code>ddeffl</code> ;</p>
</li>
<li>
<p><strong>Conséquence sur l’interpolation :</strong></p>
<ul>
<li>
<p>Pour un <strong>conjoint (<code>situat == '08'</code>)</strong> : interpolation linéaire mensuelle sur <strong><code>tarif_conj_k_new</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>) ou <strong><code>tarif_conj_k</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>&lt;</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} &lt; \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.72243em; vertical-align: -0.0391em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>).</p>
</li>
<li>
<p>Pour un <strong>orphelin (<code>situat == '09'</code>)</strong> : interpolation linéaire mensuelle sur <strong><code>tarif_orph_k_new</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>) ou <strong><code>tarif_orph_k</code></strong> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>&lt;</mo><mtext>’2022-01-01’</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} &lt; \text{'2022-01-01'}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.72243em; vertical-align: -0.0391em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">’2022-01-01’</span></span></span></span></span></span>).</p>
</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Cas réversion versée en Rente (<code>ctdver == '02'</code>) :</strong></p>
<ul>
<li>
<p><code>method = 4</code>, <code>date_deb = denai</code>, <code>date_fin = d_sim</code>, <code>annee_tx = year(d_sim)</code> ;</p>
</li>
<li>
<p>Application de <strong><code>tarif_conj_r</code></strong> (<code>ppure_imm</code>, rente viagère immédiate sur la tête du conjoint) pour <code>situat == '08'</code>, et de <strong><code>tarif_orph_r</code></strong> (rente temporaire jusqu’à 21 ans) pour <code>situat == '09'</code>.</p>
</li>
</ul>
</li>
</ul>
<h3 id="annexes--tableau-récapitulatif-vérifié-calcul_pm.py">Annexes : Tableau récapitulatif vérifié (<code>calcul_PM.py</code>)</h3>

<table>
<thead>
<tr>
<th>SITUAT</th>
<th>ctdver</th>
<th>annee_tx (jointure)</th>
<th>date_fin (avec <code>date_deb = denai</code>)</th>
<th>method</th>
<th>Tarif retenu et règle d’interpolation dans <code>interpol_taux_pm</code></th>
</tr>
</thead>
<tbody>
<tr>
<td><code>01</code>, <code>02</code></td>
<td><code>01</code></td>
<td><code>year(d_sim)</code></td>
<td><code>d_sim</code></td>
<td><code>3</code></td>
<td><code>tarif_aff_r</code> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>age_ret</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{age\_ret}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret</span></span></span></span></span></span> ; sinon <code>tarif_aff_k_new</code> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>) ou <code>tarif_aff_k</code> (avec <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_m</mtext><mo>=</mo><mn>0</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_m} = 0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span>)</td>
</tr>
<tr>
<td><code>01</code>, <code>02</code></td>
<td><code>02</code></td>
<td><code>year(d_sim)</code></td>
<td><code>d_sim</code></td>
<td><code>3</code></td>
<td><code>tarif_aff_r</code> (avec <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_m</mtext><mo>=</mo><mn>0</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_m} = 0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span>)</td>
</tr>
<tr>
<td><code>03</code></td>
<td><code>01</code></td>
<td><code>year(ddeffl)</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>tarif_aff_k_new</code> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>) ou <code>tarif_aff_k</code>, interpolé mensuellement via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>03</code></td>
<td><code>02</code></td>
<td><code>year(d_sim)</code></td>
<td><code>d_sim</code></td>
<td><code>4</code></td>
<td><code>tarif_aff_r</code></td>
</tr>
<tr>
<td><code>04</code></td>
<td><code>01</code> / <code>02</code></td>
<td><code>year(ddesh)</code></td>
<td><code>ddesh</code></td>
<td><code>1</code></td>
<td><code>tarif_dcd</code> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>rev_imm</mtext><mo>+</mo><mtext>orph_imm</mtext></mrow><annotation encoding="application/x-tex">\text{rev\_imm} + \text{orph\_imm}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">rev_imm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">orph_imm</span></span></span></span></span></span>), interpolé via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>05</code>, <code>07</code></td>
<td><code>01</code> / <code>02</code></td>
<td><code>NULL</code></td>
<td><code>NULL</code></td>
<td><code>0</code> (<code>NULL</code>)</td>
<td><code>0</code> (droits portés par les lignes réversataires <code>08</code> et <code>09</code>)</td>
</tr>
<tr>
<td><code>06</code></td>
<td><code>01</code></td>
<td><code>year(ddeffl)</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>tarif_aff_r</code> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age</mtext><mo>&lt;</mo><mtext>age_ret</mtext></mrow><annotation encoding="application/x-tex">\text{age} &lt; \text{age\_ret}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.73354em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">age</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret</span></span></span></span></span></span> ; sinon <code>tarif_aff_k_new</code> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>) ou <code>tarif_aff_k</code>, interpolé via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>06</code></td>
<td><code>02</code></td>
<td><code>year(ddesh)</code> (ou <code>-1</code>)</td>
<td><code>ddesh</code></td>
<td><code>1</code></td>
<td><code>tarif_dcd</code> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>rev_imm</mtext><mo>+</mo><mtext>orph_imm</mtext></mrow><annotation encoding="application/x-tex">\text{rev\_imm} + \text{orph\_imm}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">rev_imm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">orph_imm</span></span></span></span></span></span>), interpolé via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>08</code></td>
<td><code>01</code></td>
<td><code>year(ddeffl)</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>tarif_conj_k_new</code> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>) ou <code>tarif_conj_k</code>, interpolé mensuellement via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>08</code></td>
<td><code>02</code></td>
<td><code>year(d_sim)</code></td>
<td><code>d_sim</code></td>
<td><code>4</code></td>
<td><code>tarif_conj_r</code> (<code>ppure_imm</code>)</td>
</tr>
<tr>
<td><code>09</code></td>
<td><code>01</code></td>
<td><code>year(ddeffl)</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>tarif_orph_k_new</code> (si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8193em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>) ou <code>tarif_orph_k</code>, interpolé mensuellement via <code>mat_tx_m</code></td>
</tr>
<tr>
<td><code>09</code></td>
<td><code>02</code></td>
<td><code>year(d_sim)</code></td>
<td><code>d_sim</code></td>
<td><code>4</code></td>
<td><code>tarif_orph_r</code></td>
</tr>
</tbody>
</table><h2 id="v.-choix-de-la-colonne-critères-dinterpolation-et-points-de-vigilance">V. Choix de la colonne, critères d’interpolation et points de vigilance</h2>
<h3 id="logique-dinterpolation-linéaire-compute_interpolation_lineaire_tarif">1. Logique d’interpolation linéaire (<code>compute_interpolation_lineaire_tarif</code>)</h3>
<p>Toutes les branches de <code>interpol_taux_pm</code> (pour l’exercice <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span>) et de <code>interpol_taux_pm1</code> (pour l’exercice <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N-1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span>) reposent sur la fonction <code>compute_interpolation_lineaire_tarif(tarif, tarif_p1, mat_tx_m)</code> :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>taux_pm</mtext><mo>=</mo><mtext>tarif</mtext><mo>×</mo><mfrac><mrow><mn>12</mn><mo>−</mo><mtext>mat_tx_m</mtext></mrow><mn>12.0</mn></mfrac><mo>+</mo><mtext>tarif_p1</mtext><mo>×</mo><mfrac><mtext>mat_tx_m</mtext><mn>12.0</mn></mfrac></mrow><annotation encoding="application/x-tex">
\text{taux\_pm} = \text{tarif} \times \frac{12 - \text{mat\_tx\_m}}{12.0} + \text{tarif\_p1} \times \frac{\text{mat\_tx\_m}}{12.0}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.77777em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">tarif</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.03044em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.34444em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12.0</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.7em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord text"><span class="mord">mat_tx_m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">tarif_p1</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.00108em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.31508em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12.0</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.7em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord text"><span class="mord">mat_tx_m</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></span></p>
<p>Contrairement à la surcote (<code>update_data_pm_s</code>), <code>taux_pm</code> <strong>ne subit aucun arrondi à deux décimales</strong> après interpolation : la précision flottante (<code>double</code>) est conservée intégralement pour le calcul de la provision mathématique.</p>
<h3 id="exemple-chiffré-dinterpolation-en-capital-situation-03-ctdver--01">2. Exemple chiffré d’interpolation en capital (Situation <code>03</code>, <code>ctdver == '01'</code>)</h3>
<p>Prenons un pensionné (<code>situat = '03'</code>) né le <code>15/03/1958</code> ayant liquidé ses droits en capital (<code>ctdver = '01'</code>) à effet du <code>01/10/2023</code> (<code>DDEFFL = 2023-10-01</code>) :</p>
<ol>
<li>
<p><strong>Maturité (<code>method = 1</code>) :</strong> Entre le <code>15/03/1958</code> et le <code>01/10/2023</code>, l’assuré est âgé de <strong>65 ans</strong> (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_y</mtext><mo>=</mo><mn>65</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_y} = 65</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">65</span></span></span></span></span>) et <strong>6 mois</strong> complets (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_m</mtext><mo>=</mo><mn>6</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_m} = 6</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">6</span></span></span></span></span>).</p>
</li>
<li>
<p><strong>Sélection du barème :</strong> Comme <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>DDEFFL</mtext><mo>=</mo><mtext>2023-10-01</mtext><mo>≥</mo><mtext>2022-01-01</mtext></mrow><annotation encoding="application/x-tex">\text{DDEFFL} = \text{2023-10-01} \ge \text{2022-01-01}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">DDEFFL</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.78041em; vertical-align: -0.13597em;"></span><span class="mord text"><span class="mord">2023-10-01</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">2022-01-01</span></span></span></span></span></span>, le programme retient le nouveau barème <code>tarif_aff_k_new</code> (<code>param_conv_kal_new</code>).</p>
</li>
<li>
<p><strong>Lecture dans <code>data_tx_prov</code> :</strong></p>
<ul>
<li>
<p>À 65 ans (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_y</mtext><mo>=</mo><mn>65</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_y} = 65</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_y</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">65</span></span></span></span></span>) : <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>tarif_aff_k_new</mtext><mo>=</mo><mn>24.79</mn></mrow><annotation encoding="application/x-tex">\text{tarif\_aff\_k\_new} = 24.79</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">tarif_aff_k_new</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">24.79</span></span></span></span></span>.</p>
</li>
<li>
<p>À 66 ans (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>mat_tx_y_p1</mtext><mo>=</mo><mn>66</mn></mrow><annotation encoding="application/x-tex">\text{mat\_tx\_y\_p1} = 66</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.95444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mat_tx_y_p1</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">66</span></span></span></span></span>) : <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>tarif_aff_k_new_p1</mtext><mo>=</mo><mn>24.02</mn></mrow><annotation encoding="application/x-tex">\text{tarif\_aff\_k\_new\_p1} = 24.02</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">tarif_aff_k_new_p1</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">24.02</span></span></span></span></span>.</p>
</li>
</ul>
</li>
<li>
<p><strong>Calcul de <code>taux_pm</code> :</strong></p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>taux_pm</mtext><mo>=</mo><mn>24.79</mn><mo>×</mo><mfrac><mrow><mn>12</mn><mo>−</mo><mn>6</mn></mrow><mn>12</mn></mfrac><mo>+</mo><mn>24.02</mn><mo>×</mo><mfrac><mn>6</mn><mn>12</mn></mfrac><mo>=</mo><mn>24.405</mn></mrow><annotation encoding="application/x-tex">
\text{taux\_pm} = 24.79 \times \frac{12 - 6}{12} + 24.02 \times \frac{6}{12} = 24.405
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.72777em; vertical-align: -0.08333em;"></span><span class="mord">24.79</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.00744em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.32144em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mord">6</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.72777em; vertical-align: -0.08333em;"></span><span class="mord">24.02</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 2.00744em; vertical-align: -0.686em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.32144em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">12</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord">6</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"><span class=""></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">24.405</span></span></span></span></span></span></p>
</li>
</ol>
<h3 id="points-de-vigilance-pour-les-évolutions-futures-de-barème">3. Points de vigilance pour les évolutions futures de barème</h3>
<ul>
<li>
<p><strong>Paramétrage de <code>date_new_tarif</code> :</strong> Si un nouveau barème de conversion en capital entre en vigueur (par exemple en 2027), il conviendra de :</p>
<ol>
<li>
<p>Ajouter les nouvelles tables de paramètres (ex. <code>param_conv_kal_2027</code>, <code>param_conjoint_2027</code>, <code>param_orphelin_2027</code>) dans <code>apply_param_duplicate_on_table_parametre</code>, <code>create_tx_conv</code> et <code>create_tx_prov</code> ;</p>
</li>
<li>
<p>Étendre la liste <code>columns_tx_prov_to_keep</code> dans <code>update_data_pm_tx</code> et <code>update_data_pm_tx1</code> ;</p>
</li>
<li>
<p>Mettre à jour <code>interpol_taux_pm</code> et <code>interpol_taux_pm1</code> en passant explicitement la date de bascule depuis <code>dict_param</code> (au lieu de laisser la valeur par défaut <code>"2022-01-01"</code>).</p>
</li>
</ol>
</li>
<li>
<p><strong>Différence entre <code>interpol_taux_pm</code> (</strong><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span><strong>) et <code>interpol_taux_pm1</code> (</strong><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N-1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span><strong>) :</strong> Dans <code>interpol_taux_pm1</code>, la condition d’âge légal pour <code>situn1 in ('01', '02')</code> retranche 1 an à l’âge courant (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>−</mo><mn>1</mn><mo>&lt;</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} - 1 &lt; \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.89333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.68354em; vertical-align: -0.0391em;"></span><span class="mord">1</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> ou (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_y</mtext><mo>−</mo><mn>1</mn><mo>=</mo><mtext>age_ret_y</mtext></mrow><annotation encoding="application/x-tex">\text{age\_y} - 1 = \text{age\_ret\_y}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.89333em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_y</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_y</span></span></span></span></span></span> et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>age_m</mtext><mo>&lt;</mo><mtext>age_ret_m</mtext></mrow><annotation encoding="application/x-tex">\text{age\_m} &lt; \text{age\_ret\_m}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8491em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_m</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">age_ret_m</span></span></span></span></span></span>)) afin d’évaluer l’atteinte de l’âge légal à la date de clôture précédente <code>d_sim1</code>.</p>
</li>
</ul>
<h2 id="vi.-application-au-calcul-des-provisions-mathématiques-pm">VI. Application au calcul des Provisions Mathématiques (PM)</h2>
<p>Le mode de versement en capital (<code>ctdver == '01'</code>) n’agit pas uniquement sur la valeur de <code>taux_pm</code> : il conditionne également la <strong>Valeur de Service (<code>vs</code>)</strong> retenue et la <strong>déduction des prestations déjà versées</strong>.</p>
<h3 id="détermination-du-mode-de-versement-add_ctdver_to_data_pm">1. Détermination du mode de versement (<code>add_ctdver_to_data_pm</code>)</h3>
<p>Après proratisation éventuelle des droits des réversataires (<span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>s_pts</mtext><mo>=</mo><mo stretchy="false">⌈</mo><mtext>s_pts</mtext><mo>×</mo><mtext>tkrev1</mtext><mi mathvariant="normal">/</mi><mn>100</mn><mo stretchy="false">⌉</mo></mrow><annotation encoding="application/x-tex">\text{s\_pts} = \lceil \text{s\_pts} \times \text{tkrev1} / 100 \rceil</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mopen">⌈</span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord text"><span class="mord">tkrev1</span></span><span class="mord">/100</span><span class="mclose">⌉</span></span></span></span></span>), la fonction <code>add_ctdver_to_data_pm</code> compare le stock de points au seuil paramétré <code>seuil_K_vs_R</code> :</p>
<ul>
<li>
<p><strong><code>ctdver = "01"</code> (Capital)</strong> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>s_pts</mtext><mo>&lt;</mo><mtext>seuil_K_vs_R</mtext></mrow><annotation encoding="application/x-tex">\text{s\_pts} &lt; \text{seuil\_K\_vs\_R}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">&lt;</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">seuil_K_vs_R</span></span></span></span></span></span> ;</p>
</li>
<li>
<p><strong><code>ctdver = "02"</code> (Rente)</strong> si <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>s_pts</mtext><mo>≥</mo><mtext>seuil_K_vs_R</mtext></mrow><annotation encoding="application/x-tex">\text{s\_pts} \ge \text{seuil\_K\_vs\_R}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.94597em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">seuil_K_vs_R</span></span></span></span></span></span>.</p>
</li>
</ul>
<h3 id="impact-sur-la-valeur-de-service-create_data_pm_vs-et-create_data_pm_n_with_vs_cst">2. Impact sur la Valeur de Service (<code>create_data_pm_vs</code> et <code>create_data_pm_N_with_VS_cst</code>)</h3>
<p>Le millésime de la Valeur de Service (<code>annee_vs</code> pour la PM projetée et <code>annee_vs_cst</code> pour la PM à VS constante) dépend directement de <code>ctdver</code> :</p>
<ul>
<li>
<p><strong>Pour les contrats liquidés en Capital (<code>situat in ('03', '06', '08', '09')</code> et <code>ctdver == '01'</code>) :</strong> <code>annee_vs</code> et <code>annee_vs_cst</code> sont fixés à <strong><code>year(ddeffl)</code></strong> (valeur de service de l’année de liquidation).</p>
</li>
<li>
<p><strong>Pour tous les autres cas (<code>ctdver == '02'</code> ou contrats non liquidés <code>01</code>, <code>02</code>, <code>04</code>) :</strong> <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>annee_vs</mtext><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>d_sim</mtext><mo stretchy="false">)</mo><mo>+</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">\text{annee\_vs} = \text{year}(\text{d\_sim}) + 1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.74056em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">annee_vs</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">d_sim</span></span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span> (et <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>annee_vs_cst</mtext><mo>=</mo><mtext>year</mtext><mo stretchy="false">(</mo><mtext>d_sim</mtext><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">\text{annee\_vs\_cst} = \text{year}(\text{d\_sim})</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">annee_vs_cst</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1.06em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">year</span></span><span class="mopen">(</span><span class="mord text"><span class="mord">d_sim</span></span><span class="mclose">)</span></span></span></span></span>).</p>
</li>
</ul>
<h3 id="déduction-des-prestations-et-formules-finales-create_data_pm_p">3. Déduction des prestations et formules finales (<code>create_data_pm_p</code>)</h3>
<p>Dans <code>create_data_pm_p</code>, les prestations versées (<code>mnt</code> pour le cumul total et <code>mnt_n</code> pour le flux de l’exercice) ne sont soustraites de la provision brute <strong>que</strong> lorsque <code>(situat in ('03', '06', '08', '09')) &amp; (indic_prest == True) &amp; (ctdver == '01')</code> :</p>
<ul>
<li>
<p><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>prestation</mtext><mo>=</mo><mtext>mnt</mtext></mrow><annotation encoding="application/x-tex">\text{prestation} = \text{mnt}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8623em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">prestation</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.61508em; vertical-align: 0em;"></span><span class="mord text"><span class="mord">mnt</span></span></span></span></span></span> (sinon <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>0</mn></mrow><annotation encoding="application/x-tex">0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span>) ;</p>
</li>
<li>
<p><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>prestation_exo</mtext><mo>=</mo><mtext>mnt_n</mtext></mrow><annotation encoding="application/x-tex">\text{prestation\_exo} = \text{mnt\_n}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">prestation_exo</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mnt_n</span></span></span></span></span></span> (sinon <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>0</mn></mrow><annotation encoding="application/x-tex">0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span>) ;</p>
</li>
<li>
<p><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>prestation_1</mtext><mo>=</mo><mtext>mnt</mtext><mo>−</mo><mtext>mnt_n</mtext></mrow><annotation encoding="application/x-tex">\text{prestation\_1} = \text{mnt} - \text{mnt\_n}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">prestation_1</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.69841em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">mnt</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">mnt_n</span></span></span></span></span></span> (sinon <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>0</mn></mrow><annotation encoding="application/x-tex">0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">0</span></span></span></span></span>).</p>
</li>
</ul>
<p>Les six colonnes de provisions mathématiques de l’exercice <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi></mrow><annotation encoding="application/x-tex">N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span></span></span></span> (ainsi que <code>pm_solde1_a1</code> pour l’exercice <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow><annotation encoding="application/x-tex">N-1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.76666em; vertical-align: -0.08333em;"></span><span class="mord mathnormal" style="margin-right: 0.10903em;">N</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span><span class="mord">1</span></span></span></span></span>) sont ensuite calculées par produit du stock de points, de la valeur de service, du coefficient <code>taux_pm</code> et de la <code>surcote</code> :</p>
<ul>
<li>
<p><strong>PM totales (<code>pm_solde</code> et <code>pm_solde_vscst</code>)</strong> :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>pm_solde</mtext><mo>=</mo><mtext>s_pts</mtext><mo>×</mo><mtext>vs</mtext><mo>×</mo><mtext>taux_pm</mtext><mo>×</mo><mtext>surcote</mtext><mo>−</mo><mtext>prestation</mtext></mrow><annotation encoding="application/x-tex">
\text{pm\_solde} = \text{s\_pts} \times \text{vs} \times \text{taux\_pm} \times \text{surcote} - \text{prestation}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">pm_solde</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.66666em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">vs</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.69841em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">surcote</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.8623em; vertical-align: -0.19444em;"></span><span class="mord text"><span class="mord">prestation</span></span></span></span></span></span></span></p>
</li>
<li>
<p><strong>PM hors flux de l’exercice (<code>pm_solde1</code> et <code>pm_solde1_vscst</code>)</strong> :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>pm_solde1</mtext><mo>=</mo><mtext>s_pts1</mtext><mo>×</mo><mtext>vs</mtext><mo>×</mo><mtext>taux_pm</mtext><mo>×</mo><mtext>surcote</mtext><mo>−</mo><mtext>prestation_1</mtext></mrow><annotation encoding="application/x-tex">
\text{pm\_solde1} = \text{s\_pts1} \times \text{vs} \times \text{taux\_pm} \times \text{surcote} - \text{prestation\_1}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1.00444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">pm_solde1</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.95444em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts1</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.66666em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">vs</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.69841em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">surcote</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">prestation_1</span></span></span></span></span></span></span></p>
</li>
<li>
<p><strong>PM sur flux de l’exercice (<code>pm_exo</code> et <code>pm_exo_vscst</code>)</strong> :</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mtext>pm_exo</mtext><mo>=</mo><mtext>s_pts_exo</mtext><mo>×</mo><mtext>vs</mtext><mo>×</mo><mtext>taux_pm</mtext><mo>×</mo><mtext>surcote</mtext><mo>−</mo><mtext>prestation_exo</mtext></mrow><annotation encoding="application/x-tex">
\text{pm\_exo} = \text{s\_pts\_exo} \times \text{vs} \times \text{taux\_pm} \times \text{surcote} - \text{prestation\_exo}
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.74056em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">pm_exo</span></span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">s_pts_exo</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.66666em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">vs</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.92508em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">taux_pm</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">×</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.69841em; vertical-align: -0.08333em;"></span><span class="mord text"><span class="mord">surcote</span></span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 0.97786em; vertical-align: -0.31em;"></span><span class="mord text"><span class="mord">prestation_exo</span></span></span></span></span></span></span></p>
</li>
</ul>

