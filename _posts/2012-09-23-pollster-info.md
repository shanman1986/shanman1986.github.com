---
layout: d3
title: Pollster Info
---

<script>
	var API_SERVER = 'http://elections.huffingtonpost.com/',
		API_BASE = 'pollster/api/',
		API_FILE = 'polls.json',
		callback = '?callback=pollsterPoll',
		params = '',
		latest_data;

	window.pollsterPoll = function(incoming_data){
		latest_data = incoming_data;
		visualize();
	}

	$(document).ready(function() {
		$.ajax({
			url: API_SERVER + API_BASE + API_FILE + callback + params,
			dataType: 'script',
			type: 'GET',
			cache: true
		});
	});

	function visualize(){
		console.log(latest_data);
	}
</script>



<div id='attribution'>
	<p>This page uses data obtained from the Huffington Post's <a href='http://elections.huffingtonpost.com/pollster/api/'>Pollster API</a>.</p>
</div>