---


---

<h2 id="tableau-récapitulatif-final">5. Tableau récapitulatif final</h2>

<table>
<thead>
<tr>
<th>Situation</th>
<th>annee_s</th>
<th>date_deb</th>
<th>date_fin</th>
<th>method</th>
<th>Paire (surcote/surcotenv)</th>
<th>Post-traitement v9</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>01</strong></td>
<td><code>year(d_sim)</code></td>
<td><code>denai</code></td>
<td><code>d_sim</code></td>
<td><code>indic_mat_surcote_aff</code></td>
<td><code>surcotenv</code></td>
<td>✅ SAS 1961–1967</td>
</tr>
<tr>
<td><strong>02</strong></td>
<td><code>year(d_sim)</code></td>
<td><code>denai</code></td>
<td><code>d_sim</code></td>
<td><code>indic_mat_surcote_aff</code></td>
<td><code>surcotenv</code></td>
<td>✅ SAS 1961–1967</td>
</tr>
<tr>
<td><strong>03</strong></td>
<td><code>year(ddeffl)</code></td>
<td><code>denai</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>ddeffl &lt; 2015-03-01</code> ? <code>surcote</code> : <code>surcotenv</code></td>
<td>❌</td>
</tr>
<tr>
<td><strong>04</strong></td>
<td><code>year(ddesh)</code> si <code>ddrth ≤ dedcd</code> sinon <code>year(ddrth)</code></td>
<td><code>denai</code></td>
<td><code>ddesh</code> ou <code>ddrth</code></td>
<td><code>1</code></td>
<td><code>dedcd &lt; 2015-03-01</code> → <code>surcote</code> (sous-conditions), sinon <code>surcotenv</code></td>
<td>❌</td>
</tr>
<tr>
<td><strong>05</strong></td>
<td>idem 04</td>
<td><code>denai</code></td>
<td>idem 04</td>
<td><code>1</code></td>
<td>idem 04</td>
<td>❌</td>
</tr>
<tr>
<td><strong>06</strong></td>
<td><code>year(ddeffl)</code></td>
<td><code>denai</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td><code>ddeffl &lt; 2015-03-01</code> ? <code>surcote</code> : <code>surcotenv</code></td>
<td>❌</td>
</tr>
<tr>
<td><strong>07</strong></td>
<td><code>year(ddeffl)</code></td>
<td><code>denai</code></td>
<td><code>ddeffl</code></td>
<td><code>1</code></td>
<td>idem 06</td>
<td>❌</td>
</tr>
<tr>
<td><strong>08</strong></td>
<td><code>NULL</code></td>
<td>—</td>
<td>—</td>
<td>—</td>
<td>forcée <code>1</code> → remplacée par <code>surcote_ad</code> (adhérent principal)</td>
<td>❌</td>
</tr>
<tr>
<td><strong>09</strong></td>
<td><code>NULL</code></td>
<td>—</td>
<td>—</td>
<td>—</td>
<td>idem 08</td>
<td>❌</td>
</tr>
<tr>
<td><strong>X, Y</strong></td>
<td><code>NULL</code></td>
<td>—</td>
<td>—</td>
<td>—</td>
<td>forcée <code>1</code>, non remplacée</td>
<td>❌</td>
</tr>
</tbody>
</table>
