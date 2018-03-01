---
title: 附近的人、商家，计算两点之间的距离
date: 2017-12-02 10:57:42
tags: PHP

---
	
	define('EARTH_RADIUS', 6371);//地球半径，平均半径为6371km
	//获取该点周围的4个点
	$distance = 1;//范围（单位千米）
	$lat = 113.873643;
	$lng = 22.573969;
	$dlng = 2 * asin(sin($distance / (2 * EARTH_RADIUS)) / cos(deg2rad($lat)));
	$dlng = rad2deg($dlng);
	$dlat = $distance/EARTH_RADIUS;
	$dlat = rad2deg($dlat);
	$squares = array('left-top'=>array('lat'=>$lat + $dlat,'lng'=>$lng-$dlng),
	        'right-top'=>array('lat'=>$lat + $dlat, 'lng'=>$lng + $dlng),
	        'left-bottom'=>array('lat'=>$lat - $dlat, 'lng'=>$lng - $dlng),
	        'right-bottom'=>array('lat'=>$lat - $dlat, 'lng'=>$lng + $dlng)
	        );

	//从数库查询匹配的记录
	$info_sql = "SELECT * FROM `数据表` WHERE (`lat` BETWEEN $squares['right-bottom']['lat'] AND $squares['left-top']['lat']) AND (`lng` BETWEEN $squares['right-bottom']['lng'] AND $squares['left-top']['lng'])";

	//获取两点之间的距离
	function getDistanceBetweenPointsNew($latitude1, $longitude1, $latitude2, $longitude2) {
	  $theta = $longitude1 - $longitude2;
	  $miles = (sin(deg2rad($latitude1)) * sin(deg2rad($latitude2))) + (cos(deg2rad($latitude1)) * cos(deg2rad($latitude2)) * cos(deg2rad($theta)));
	  $miles = acos($miles);
	  $miles = rad2deg($miles);
	  $miles = $miles * 60 * 1.1515;
	  $feet = $miles * 5280;
	  $yards = $feet / 3;
	  $kilometers = $miles * 1.609344;
	  $meters = $kilometers * 1000;
	  return compact('miles','feet','yards','kilometers','meters'); 
	}

	$point1 = array('lat' => 40.770623, 'long' => -73.964367);
	$point2 = array('lat' => 40.758224, 'long' => -73.917404);
	$distance = getDistanceBetweenPointsNew($point1['lat'], $point1['long'], $point2['lat'], $point2['long']);
	foreach ($distance as $unit => $value) {
	  echo $unit.': '.number_format($value,4).'<br />';
	}


转自 http://www.jb51.net/article/83974.htm
