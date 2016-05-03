## 地址格式

<table>
	<tbody>
		<tr>
			<td>格式号</td><td>格式规则</td><td>默认影响国家</td><td>示例</td>
		</tr>
		<tr>
			<td>1</td>
			<td><pre>$firstname $lastname
$streets
$city, $state   $postcode
$country
</pre></td>
			<td>所有以下未提及国家</td>
			<td><pre>James Logan
496 Victoria Street 
Sydney, 2010
New South Wales, Australia
</pre></td>
		</tr>
		<tr>
			<td>2</td>
			<td><pre>$firstname $lastname
$streets
$city, $state   $postcode
$country
</pre></td>
			<td>United States</td>
			<td><pre>John Doe
123 Magnolia Street
Dallas, TX   55803-0034
United States
</pre></td>
		</tr>
		<tr>
			<td>3</td>
			<td><pre>$firstname $lastname
$streets
$city
$postcode - $state, $country
</pre></td>
			<td>Spain</td>
			<td><pre>Martina Gonzalez
General Castanos, 86 
Barcelona
08003 - Barcelona, Spain

* 提示：上面的城市和州都为Barcelona，西班牙使用省的称呼代替州
</pre></td>
		</tr>
		<tr>
			<td>4</td>
			<td><pre>$firstname $lastname
$streets
$city ($postcode)
$country
</pre></td>
			<td>Singapore</td>
			<td><pre>John Low
16 Whampoa Drive
Singapore (260042)
Singapore
</pre></td>
		</tr>
		<tr>
			<td>5</td>
			<td><pre>$firstname $lastname
$streets
$postcode $city
$country
</pre></td>
			<td>Austria, Germany</td>
			<td><pre>Heidi Kohler
Lentzeallee 194 
14195 Berlin
Germany
</pre></td>
		</tr>
		<tr>
			<td>6</td>
			<td><pre>$firstname $lastname
$streets
$city
$state
$postcode
$country
</pre></td>
			<td>United Kingdom</td>
			<td><pre>James Watson
1 St. Georges Business Centre
Portsmouth
PO1 3AX
United Kingdom

* 提示：上面的城市和州使用了相同的Portsmouth，在此处会省略 
</pre></td>
		</tr>
	</tbody>
</table>

