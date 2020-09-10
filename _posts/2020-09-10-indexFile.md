---
layout: post
title:  "Game detection program ServerCode"
date:   2020-09-10 12:10:00 +0700
categories: [HTML, JavaScript, jQuery]
---

### NodeJs가 연결한 index.ejs

**코드**

```html
<!DOCTYPE html>
<html>
<head>
<style>
   .all {
	  background-color: #B0E8F1;
	  color: black;
	  margin: 20px;
	  padding: 10px;
	}
	.check {
	  background-color: #7CD0D5;
	  color: black;
	  margin: 20px;
	  padding: 10px;
	  border: 2px solid black;
	}
   	table {
   	  width:100%;
	}
	table, th, td {
  	  border: 1px solid black;
  	  border-collapse: collapse;
	}
	th, td {
  	  padding: 15px;
  	  text-align: center;
	}
	#detectGame tr:nth-child(even) {
  	  background-color: #eee;
	}

	#detectGame tr:nth-child(odd) {
 	  background-color: #fff;
	}

	#detectGame th {
  	  background-color: black;
  	  color: white;
	}
</style>
   </head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.3.0/socket.io.js"></script>
<script src="http://code.jquery.com/jquery-1.11.1.js"></script>
<script type="text/javascript"
src="https://code.jquery.com/jquery-2.1.0.min.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function() {

        // 시간을 딜레이 없이 나타내기위한 선 실행
        realTimer();
        // 이후 0.5초에 한번씩 시간을 갱신한다.
        setInterval(realTimer, 500);
    });

    function realTimer() {
		const nowDate = new Date();
		const year = nowDate.getFullYear();
		const month= nowDate.getMonth() + 1;
		const date = nowDate.getDate();
		const hour = nowDate.getHours();
		const min = nowDate.getMinutes();
		const sec = nowDate.getSeconds();
		document.getElementById("nowTimes").innerHTML = 
                  year + " / " + addzero(month) + " /  " + addzero(date) + "&nbsp;" + hour + ":" + addzero(min) + ":" + addzero(sec);
	}

	function addzero(num) {
		if(num < 10) { num = "0" + num; }
 		return num;
	}
	var idArr = new Array();
	function deleteLine(obj) {
    	var tr = $(obj).parent().parent();
    	tr.remove();
	}
	$(() => {
	    const socket = io('http://59.12.245.249:30002');	
			socket.on('reqMsg', (id, status, check) => {
				$(function(){
					var row = $('#detectGame tbody tr').length;
					if(status == "Yes"){
						if(!idArr.includes(id)) {
							idArr[row] = id;
							$(".checking").text(check);
							$('#detectGame > tbody:last').append('<tr><td class="id">' + id + '</td><td class="status">' + "!" + '</td></tr>')
						} else {
							$(".checking").text(check);
						}
					} else {
						$(".checking").text(check);
						if(idArr.includes(id)) {
							$('#detectGame tbody tr').remove();
							idArr.splice(idArr.indexOf(id));
						}
					}
    			});
		});
	});
</script>

<body>
	<div class="all">
		<h1><span id="nowTimes"></span></h1>
		<div class="check">
			<h2> 미접속 학생 번호 </h2>
			<h3 class="checking"></h3>
		</div>
	</div>
	<table id="detectGame">
	  <thead>	
        <tr>
           <th> 학번 </th>
           <th> 상태 </th>
        </tr>
      </thead>
      <tbody>
  	  </tbody>   
</body>
</html>
````