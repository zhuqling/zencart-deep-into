## 地址格式

<table>
	<tbody>
		<tr>
			<td>格式号</td><td>格式规则</td><td>默认影响国家</td><td>示例</td>
		</tr>
		<tr>
			<td>1</td><td>$firstname $lastname<br /> $streets<br /> $city, $postcode$state,$country</td><td>所有以下未提及国家</td><td>James Logan496 Victoria StreetSydney, 2010New South Wales, Australia</td>
		</tr>
		<tr>
			<td>2</td><td>$firstname $lastname$streets$city, $state&nbsp;&nbsp; $postcode$country</td><td>United States</td><td>John Doe123 Magnolia StreetDallas, TX&nbsp;&nbsp; 55803-0034United States</td>
		</tr>
		<tr>
			<td>3</td><td>$firstname $lastname$streets$city$postcode - $state, $country</td><td>Spain</td><td>Martina GonzalezGeneral Castanos, 86Barcelona08003 - Barcelona, Spain提示：上面的城市和州都为Barcelona，西班牙使用省的称呼代替州</td>
		</tr>
		<tr>
			<td>4</td><td>$firstname $lastname$streets$city ($postcode)$country</td><td>Singapore</td><td>ohn Low16 Whampoa DriveSingapore (260042)Singapore</td>
		</tr>
		<tr>
			<td>5</td><td>$firstname $lastname$streets$postcode $city$country</td><td>Austria, Germany</td><td>Heidi KohlerLentzeallee 19414195 BerlinGermany</td>
		</tr>
		<tr>
			<td>6</td><td>$firstname $lastname$streets$city$state$postcode$country</td><td>United Kingdom</td><td>James Watson1 St. Georges Business CentrePortsmouthPO1 3AXUnited Kingdom提示：上面的城市和州使用了相同的Portsmouth，在此处会省略</td>
		</tr>
	</tbody>
</table>

> 提示：上面的城市和州使用了相同的Portsmouth，在此处会省略 
