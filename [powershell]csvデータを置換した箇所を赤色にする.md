---


---

<h2 id="powershellcsvの特定文字を置換し、置換箇所を赤色にしたhtmlを出力するスクリプト">[powershell]csvの特定文字を置換し、置換箇所を赤色にしたhtmlを出力するスクリプト</h2>
<pre class=" language-powershell"><code class="prism  language-powershell"><span class="token comment">##########################################################</span>
<span class="token comment">#郵便番号データから川を含む県を抽出し、県名、市町村名を</span>
<span class="token comment">#川-&gt;山に置換したものをcsvとhtmlで出力するスクリプト。</span>
<span class="token comment">#htmlは置換箇所を赤色にする</span>
<span class="token comment">##########################################################</span>

<span class="token comment">#郵便番号情報をWEBから取得</span>
<span class="token function">write-host</span> <span class="token string">"郵便番号データ取得"</span>
<span class="token function">Invoke-WebRequest</span> <span class="token string">"https://www.post.japanpost.jp/zipcode/dl/kogaki/zip/ken_all.zip"</span> <span class="token operator">-</span>OutFile <span class="token string">"ken_all.zip"</span>

<span class="token comment">#データがある場合削除</span>
<span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">test-path</span> <span class="token string">".\ken_all.csv"</span><span class="token punctuation">)</span><span class="token punctuation">{</span><span class="token function">remove-item</span> <span class="token string">"ken_all.csv"</span><span class="token punctuation">}</span>

<span class="token function">write-host</span> <span class="token string">"解凍"</span>
Expand<span class="token operator">-</span>Archive <span class="token string">"ken_all.zip"</span> <span class="token operator">-</span>DestinationPath <span class="token string">".\"</span>
<span class="token comment">#解凍前ファイル削除</span>
<span class="token function">remove-item</span>  <span class="token string">"ken_all.zip"</span>

<span class="token comment">#CSVデータの読み込み、ヘッダは適当にa～oで設定</span>
<span class="token function">write-host</span> <span class="token string">"csvのロード取得"</span>
<span class="token variable">$csv</span> = <span class="token function">import-csv</span> <span class="token operator">-</span>Encoding default <span class="token operator">-</span>header a<span class="token punctuation">,</span>b<span class="token punctuation">,</span>c<span class="token punctuation">,</span>d<span class="token punctuation">,</span>e<span class="token punctuation">,</span>f<span class="token punctuation">,</span>g<span class="token punctuation">,</span>h<span class="token punctuation">,</span>I<span class="token punctuation">,</span>j<span class="token punctuation">,</span>k<span class="token punctuation">,</span>l<span class="token punctuation">,</span>m<span class="token punctuation">,</span>n<span class="token punctuation">,</span>o <span class="token string">"KEN_ALL.CSV"</span> 

<span class="token comment">#CSV出力ファイル名</span>
<span class="token variable">$script</span>:outfile = @<span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token comment">#HTML出力ファイル名</span>
<span class="token variable">$script</span>:outfile2 = @<span class="token punctuation">(</span><span class="token punctuation">)</span>

<span class="token function">write-host</span> <span class="token string">"csvデータを一件ずつ処理"</span>
<span class="token variable">$csv</span> <span class="token punctuation">|</span> ? <span class="token punctuation">{</span><span class="token variable">$_</span><span class="token punctuation">.</span>g <span class="token operator">-like</span> <span class="token string">"*川*"</span><span class="token punctuation">}</span><span class="token punctuation">|</span> <span class="token function">select</span> g<span class="token punctuation">,</span>h <span class="token punctuation">|</span>  <span class="token function">Sort-Object</span> <span class="token operator">-</span>Unique g<span class="token punctuation">,</span>h <span class="token punctuation">|</span> <span class="token operator">%</span><span class="token punctuation">{</span>
    <span class="token variable">$r</span>=<span class="token variable">$_</span>
    <span class="token variable">$r2</span>=<span class="token variable">$r</span> <span class="token punctuation">|</span> <span class="token function">select</span> <span class="token operator">*</span>  <span class="token comment">#そのまま渡すと参照渡しになるのでselectをかけている</span>

    <span class="token comment">#g列(県名)とh(市町村)列をそれぞれ山-&gt;川に置換</span>
    <span class="token variable">$r2</span><span class="token punctuation">.</span>g = <span class="token variable">$r2</span><span class="token punctuation">.</span>g <span class="token operator">-replace</span> <span class="token string">"川"</span><span class="token punctuation">,</span><span class="token string">"山"</span>
    <span class="token variable">$r2</span><span class="token punctuation">.</span>h = <span class="token variable">$r2</span><span class="token punctuation">.</span>h <span class="token operator">-replace</span> <span class="token string">"川"</span><span class="token punctuation">,</span><span class="token string">"山"</span>
    <span class="token variable">$redString</span> = <span class="token variable">$NUll</span>

    <span class="token punctuation">(</span><span class="token string">"g"</span><span class="token punctuation">,</span><span class="token string">"h"</span><span class="token punctuation">)</span> <span class="token punctuation">|</span> <span class="token operator">%</span><span class="token punctuation">{</span>
      <span class="token comment">#char配列に変換</span>
      <span class="token variable">$s</span> = <span class="token punctuation">(</span><span class="token variable">$r</span><span class="token punctuation">.</span><span class="token punctuation">(</span><span class="token variable">$_</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">.</span>tochararray<span class="token punctuation">(</span><span class="token punctuation">)</span>
      <span class="token variable">$s2</span> = <span class="token punctuation">(</span><span class="token variable">$r2</span><span class="token punctuation">.</span><span class="token punctuation">(</span><span class="token variable">$_</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">.</span>tochararray<span class="token punctuation">(</span><span class="token punctuation">)</span>

      <span class="token comment">#1文字ずつ比較(置換前後で文字数が変わらない前提のロジック)</span>
      <span class="token punctuation">(</span>0<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">(</span><span class="token variable">$s</span><span class="token punctuation">.</span>length <span class="token operator">-</span> 1<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">|</span> <span class="token operator">%</span><span class="token punctuation">{</span>
          <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token variable">$s</span><span class="token punctuation">[</span><span class="token variable">$_</span><span class="token punctuation">]</span> <span class="token operator">-ne</span> <span class="token variable">$s2</span><span class="token punctuation">[</span><span class="token variable">$_</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
              <span class="token comment">#差異がある場合&lt;font&gt;タグで赤色にする</span>
              <span class="token variable">$redString</span> = <span class="token variable">$redString</span> <span class="token operator">+</span> <span class="token string">"&lt;font color=red&gt;"</span> <span class="token operator">+</span> <span class="token variable">$s2</span><span class="token punctuation">[</span><span class="token variable">$_</span><span class="token punctuation">]</span> <span class="token operator">+</span> <span class="token string">"&lt;/font&gt;"</span>   
          <span class="token punctuation">}</span><span class="token keyword">else</span><span class="token punctuation">{</span>
              <span class="token variable">$redString</span> = <span class="token variable">$redString</span> <span class="token operator">+</span> <span class="token variable">$s</span><span class="token punctuation">[</span><span class="token variable">$_</span><span class="token punctuation">]</span>
          <span class="token punctuation">}</span>
      <span class="token punctuation">}</span>
      <span class="token variable">$r2</span><span class="token punctuation">.</span><span class="token punctuation">(</span><span class="token variable">$_</span><span class="token punctuation">)</span> = <span class="token variable">$redString</span>
    <span class="token punctuation">}</span>

    <span class="token variable">$script</span>:outfile = <span class="token variable">$outfile</span> <span class="token operator">+</span> <span class="token variable">$r</span>
    <span class="token variable">$script</span>:outfile2 = <span class="token variable">$outfile2</span> <span class="token operator">+</span> <span class="token variable">$r2</span>
<span class="token punctuation">}</span>

<span class="token comment">#System.Webクラスロード</span>
<span class="token namespace">[System.Reflection.Assembly]</span>::LoadWithPartialName<span class="token punctuation">(</span><span class="token string">"System.Web"</span><span class="token punctuation">)</span> <span class="token punctuation">|</span> <span class="token function">out-null</span>
<span class="token comment">#ConvertTo-Html するとhtml特殊文字がエスケープされるので元の値に戻す</span>
<span class="token variable">$script</span>:outfile2 <span class="token punctuation">|</span> <span class="token function">ConvertTo-Html</span> <span class="token punctuation">|</span> <span class="token operator">%</span><span class="token punctuation">{</span><span class="token namespace">[System.Web.HttpUtility]</span>::HtmlDecode<span class="token punctuation">(</span><span class="token variable">$_</span><span class="token punctuation">)</span> <span class="token punctuation">}</span> <span class="token punctuation">|</span> <span class="token function">out-file</span> <span class="token string">"out.html"</span>
<span class="token comment">#オリジナルデータ内にも特殊文字が含まれる場合は以下のように固定値を置換する</span>
<span class="token comment">#$script:outfile2 | ConvertTo-Html |</span>
<span class="token comment"># %{($_.replace("&amp;lt;font color=red&amp;gt;","&lt;font color=red&gt;")).replace("&amp;lt;/font&amp;gt;","&lt;/font&gt;") } |</span>
<span class="token comment">#  out-file "out\out.html"</span>

<span class="token comment">#CSVに変換</span>
<span class="token variable">$script</span>:outfile <span class="token punctuation">|</span> <span class="token function">ConvertTo-csv</span> <span class="token operator">-</span>NoTypeInformation <span class="token punctuation">|</span> <span class="token function">out-file</span> <span class="token operator">-</span>encoding default <span class="token string">"out.csv"</span>

</code></pre>

