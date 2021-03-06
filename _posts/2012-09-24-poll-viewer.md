---
layout: polls
---

#Pollster API: Poll Viewer

This tool will let you check the results of a request to the [Pollster API](http://elections.huffingtonpost.com/pollster/api) quickly and easily.

Enter your parameters below to view the polls returned from the API:

<code>http://elections.huffingtonpost.com/pollster/api/polls.json?</code><input id='input-params' type='text' /><input type='submit' value='fetch' onclick='fetch()' />

<div id='output'></div>

<script>

	String.prototype.makeCal = function(){
		var output = this.replace(/(([A-Za-z]){3}) (([A-Za-z]){3}) (([0-9]){2}) (([0-9]){4})/g, "<span class='cal-cal'><span class='cal-month'>$3 $7</span><span class='cal-date'>$5</span></span>");
		return output;
	}

	var graph_height = 120,
		bar_width = 30,
		bar_padding = 36,
		label_padding = 16;

	var scaleYUp = d3.scale.linear()
		.domain([0,100])
		.range([graph_height, 0]);
	var scaleYDn = d3.scale.linear()
		.domain([0,100])
		.range([0, graph_height]);

	var API_SERVER = 'http://elections.huffingtonpost.com/',
		API_BASE = 'pollster/api/',
		API_FILE = 'polls.json',
		callback = '?callback=pollsterPoll',
		latest_data;

	var format = d3.time.format('%Y-%m-%d');

	window.pollsterPoll = function(incoming_data){
		latest_data = incoming_data;
		visualize();
	}

	function fetch(){
		var input_params = document.getElementById('input-params').value; 
		$.ajax({
			url: API_SERVER + API_BASE + API_FILE + callback + '&' + input_params,
			dataType: 'script',
			type: 'GET',
			cache: true
		});
	}

	function visualize(){

		//clear old view
		d3.select('#output').selectAll('.poll-box-wrapper').remove();

		var view = d3.select('#output');
		var boxes = view.selectAll('.poll-box').data(latest_data);

		var boxEnter = boxes.enter().append('div')
			.attr('class', 'poll-box-wrapper')
			.html(function(d,i){
				var counter = '<h3 class="response-obj">response[' + i + ']</h3>';
				return counter;
			})
				.append('div')
					.attr('class', 'poll-box')
					.html(function(d,i){
						var start = '<span class="cat-title">start_date:</span> ' + format.parse(d.start_date).toDateString().makeCal(),
							end = ' <span class="cat-title">end_date:</span> ' + format.parse(d.end_date).toDateString().makeCal() + '<br />',
							method = '<span class="cat-title">method:</span> <span class="cat-method">' + d.method + '</span><br /><br />',
							pollster = '<span class="cat-title">pollster:</span> <span class="cat-pollster">' + d.pollster + '</span><br />',
							wrapper = '<div class="questions-wrapper"></div>',
							source = '<div class="cat-source">source: <a href="' + d.source + '">' + d.source + '</a></div>';

						return pollster + method + start + end + wrapper + source;
					})
					.select('.questions-wrapper').selectAll('.question').data(function(d){return d.questions})
						.enter().append('div')
							.attr('class', 'question')
							.html(function(d,i){
								var question_number = i + 1,
									question = d.name,
									header = '<span class="quest-num">Question ' + question_number + ':</span> ' + question;
								return '<header>' + header + '</header>'
							})
							.selectAll('.subpop').data(function(d){return d.subpopulations})
								.enter().append('div')
									.attr('class', 'subpop')
									.html(function(d){
										var header = 'Sample: <span class="obs-num">'+ d.observations + '</span> ' + d.name;
										return '<header>' + header + '</header>';
									});

		var graphEnter = boxEnter.append('svg:svg')
			.attr('class', 'response-vis')
			.attr('width', function(d){return d.responses.length * (bar_width + (bar_padding*2))})
			.attr('height', graph_height + label_padding)
			.selectAll('.response').data(function(d){return d.responses})
				.enter().append('svg:g')
					.attr('class', 'response');

		var responseBarEnter = graphEnter
						.insert('svg:rect')
						.attr('class', 'response-bar')
						.attr('x', function(d,i){return bar_padding + (i * bar_width) + (2 * i * bar_padding)})
						.attr('y', function(d){return scaleYUp(d.value)})
						.attr('height', function(d){return scaleYDn(d.value)})
						.attr('width', bar_width)
						.attr('fill', function(d){
							if(d.party === 'Dem'){
								return 'steelblue';
							} else if(d.party === 'Rep'){
								return 'firebrick';
							} else if(d.party === 'ind'){
								return '#FD7';
							} else if(d.choice === 'Approve' || d.choice === 'Yes' || d.choice === 'Very Favorable' || d.choice === 'Favorable' || d.choice === 'Positive' || d.choice === 'Very Positive'){
								return '#0F0';
							} else if(d.choice === 'Disapprove' || d.choice === 'No' || d.choice === 'Very Unfavorable' || d.choice === 'Unfavorable' || d.choice === 'Negative' || d.choice === 'Very Negative'){
								return '#F00';
							} else if(d.choice === 'Somewhat Favorable' || d.choice === 'Somewhat Positive'){
								return '#7F7';
							} else if(d.choice === 'Somewhat Unfavorable' || d.choice === 'Somewhat Negative'){
								return '#F77';
							} else {
								return '#777';
							}
						});

		var responseValueLabelEnter = graphEnter
						.insert('svg:text')
							.attr('class', 'response-value-label')
							.attr('x', function(d,i){return bar_padding + (i * bar_width) + (2 * i * bar_padding) + (0.5 * bar_width)})
							.attr('y', function(d){return scaleYUp(d.value)})
							.attr('dy', -7)
							.attr('text-anchor', 'middle')
							.text(function(d){return d.value + '%'});

		var responseChoiceLabelEnter = graphEnter
						.insert('svg:text')
							.attr('class', 'response-choice-label')
							.attr('x', function(d,i){return bar_padding + (i * bar_width) + (2 * i * bar_padding) + (0.5 * bar_width)})
							.attr('y', graph_height)
							.attr('dy', label_padding - 5)
							.attr('text-anchor', 'middle')
							.text(function(d){return d.choice});	

	}
</script>


<div id='attribution'>Made with data from HuffPost's <a href='http://elections.huffingtonpost.com/pollster/api'>Pollster API</a>.</div>