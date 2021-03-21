Data Plotter
***************

The file "GDELT_Custom_Chart.php" contains the following features:

1. A flipclock, which shows the time since the experiment is running
2. A timer showing when the next update to the nodes is made
3. A graph showing the queried data visually

In the following, the three parts from above are briefly explained. 

Flipclock
=========

The javascript source code for the flipclock can be found |flipclock_source_code|. In the case of this project, the file was renamed to "flipclock.js" and is located in a subfolder called "js".
The css source code from |flipclock_source_css| was slighlty modified and is located in the folder "css".

.. |flipclock_source_code| raw:: html

   <a href="https://gist.github.com/danichim/8da99dbd742b7cebbaa3b20b4468f48c" target="_blank">here</a>

.. |flipclock_source_css| raw:: html

    <a href="https://github.com/objectivehtml/FlipClock/blob/master/src/flipclock/css/flipclock.css" target="_blank">here</a>


To create a clock, one simply has to create an empty div with a unique class or id, in our case the div is the following:

.. code-block:: html

    <div class='clock'></div>

To setup the clock, the following code needs to be run:

.. code-block:: javascript

    clock = $('.clock').FlipClock(difSeconds,{
		        clockFace: 'DailyCounter'
		    });

The difSeconds can be derived with:

.. code-block:: javascript

    var difSeconds = new Date().getTime()/1000;

In our case, the initial setup time needs to be removed, such that we see only the days form that day on:

.. code-block:: javascript

    var uptimeDate = JSON.parse(result).r3[0];
    var upDateTime = uptimeDate.min;
    var year =  upDateTime.substr(0,4);
    var month = upDateTime.substr(5,2);
    var date =  upDateTime.substr(8,2);
    var hour =  upDateTime.substr(11,2);
    var mint =  upDateTime.substr(14,2);
    var sec =   upDateTime.substr(17,2);
    
    //month are counted from 0 to 11 so 1 is subtracted from month
    var uptimeDTSeconds = new Date(year,month-1,date,hour,mint,sec).getTime() / 1000;
    var currentDTSeconds = new Date().getTime()/1000;
    var difSeconds = currentDTSeconds - uptimeDTSeconds;
    
    console.log('up= ',uptimeDTSeconds);
    console.log('cur= ',currentDTSeconds);
    console.log('dif= ',difSeconds);
    
    clock = $('.clock').FlipClock(difSeconds,{
        clockFace: 'DailyCounter'
    });

'result' is the json string descripbed in :ref:`Database Helper`.

Update timer
=============

The source code from the update timer can be found here: https://github.com/wimbarelds/TimeCircles/blob/master/inc/TimeCircles.js
The 'TimeCircles.js' file is again found under 'js', the 'TimeCircles.css' file under 'css'.

As with the FlipClock, a simple empty div with distinct class or id needs to be created:

.. code-block:: html

    <div class='update_timer' data-timer='900' style='width:50px;height:50px;'></div>

Then, for initializing it, the dollowing code gets executed:

.. code-block:: javascript

    $('.update_timer').TimeCircles({
    time: {
        Days: { show: false },
        Hours: { show: false },
        Minutes: { show: false },
        Seconds: { show: true,color: '#1e73be'}
    },
    text_size: 0.15,
    number_size: 0.20,
    total_duration: 900,
    circle_bg_color: '#f7f8f9',
    fg_width: 0.03
    });
    var container = $('.update_timer .textDiv_Seconds');
    container.find('h4').text('');
    var original = container.find('span');
    var clone = original.clone().appendTo(container);
    original.hide();

We also add a listener to the timer:

.. code-block:: javascript

    $('.update_timer').TimeCircles().addListener(function(unit, value, total) {
    total = Math.abs(total);
    var minutes = Math.floor(total / 60) % 60;
    var seconds = total % 60;
    if(seconds < 10) seconds = '0' + seconds;
        clone.text(minutes + ':' + seconds);
    }, 'all');

The variable 'clone' is the variable from the code block before.

This whole process creates the Timer, but with a slight adaptation:
The original seconds hand gets hidden and e replacement gets added.
Every second, the value of the real second box gets read and, if less that 10, a leading 0 gets added.

Whenever there is an error with a file not beign able to be loaded, the Timer gets restarted:

.. code-block:: javascript

    $('.update_timer').TimeCircles().restart();


Graph
=====

For the graph, the plotly library was used (https://plotly.com/javascript/).

Also for plotly, an empty div **with a unique id** needs to be created.

.. code-block:: html

	<div id='cvs'></div>

Then, the plot needs to be initiated:

.. code-block:: javascript

    Plotly.react('cvs', data, layout,{displaylogo: false})
    Plotly.setPlotConfig({
        modeBarButtonsToRemove: ['sendDataToCloud']
    });	

The display and layout variables are initiated like in the following:

.. code-block:: javascript
    
    var res = JSON.parse(result).r1;

    var rangeStartingDt = new Date((res[res.length-1].epoch-((res[res.length-1].epoch-res[0].epoch)/7)) *1000).toLocaleDateString('ko-KR');
    rangeStartingDt = ((rangeStartingDt.replace('. ','-')).replace('. ','-')).replace('.','');
    var rangeStartingTm = new Date((res[res.length-1].epoch-((res[res.length-1].epoch-res[0].epoch)/7)) *1000).toLocaleTimeString('en-GB');
    var rangeStartingTime = rangeStartingDt+' '+rangeStartingTm;

    var rangeEndingDt = new Date((res[res.length-1].epoch-((res[res.length-1].epoch-res[0].epoch)/7)) *1000).toLocaleDateString('ko-KR');
    rangeEndingDt = ((rangeEndingDt.replace('. ','-')).replace('. ','-')).replace('.','');
    var rangeEndingTime = rangeEndingDt+' '+endingTm;
    
    var xAxisTitle = startingTime+'                            to                          '+endingTime;
	var yAxisTitle = 'Total Number of Event News';

    var layout = {
        autosize: false,
        width: 1150,
        height:600,
        paper_bgcolor:'#e7e8e9',
        plot_bgcolor:'#e7e8e9',
        'titlefont': {
            'size': 12
        },
        xaxis: {
            title: xAxisTitle,
            titlefont: {
                family: 'Open Sans, verdana, arial, sans-serif',
                size: 14,
                color: '#444444'
            },
            range:[rangeStartingTime, rangeEndingTime],
            rangeslider: {},
        },
        yaxis: {
            title: yAxisTitle,
            titlefont: {
            family: 'Open Sans, verdana, arial, sans-serif',
            size: 14,
            color: '#444444'
            },
            range:[0,1000]
        },
        legend: {
                        xanchor:'left',
                        yanchor:'top',
                        y:1.5, 
                        x:0,  
                        orientation: 'h'
            },
    };

    var data = [trace1,trace2,trace3,trace4,trace5,trace6,trace7,trace8,trace9,trace10,trace11,trace12,trace13,trace14,trace15,trace16,trace17,trace18,trace19,trace20,trace21,trace22,trace23,trace24,trace25,trace26,trace27,trace28,trueSumTraces,SumStatesTraces,DiasSumTraces];


As trace1 to trace28 are almost identical, only one of them is shown in the following, plus the additional data fields:

.. code-block:: javascript

    var res = JSON.parse(result).r1;
    var countries = JSON.parse(result).r2;

    var xAxisValue<no.>=[]; //eg. xAxisValue1
    var yAxisValue<no.>=[];

    var peers=[];

    var xAxisTrueSumOfStateVals = [];
    var yAxisTrueSumOfStateVals = [];

    var xAxisDIASSumOfStateVals = [];
    var yAxisDIASSumOfStateVals = [];
    
    var xAxisSumofSelectedStateVals = [];
    var yAxisSumofSelectedStateVals = [];

    for(i=0;i<res.length;i++){
        console.log(res[i]);
        var epochValDt = new Date((res[i].epoch)*1000).toLocaleDateString('ko-KR');
        epochValDt = ((epochValDt.replace('. ','-')).replace('. ','-')).replace('.','');

        var epochValTm = new Date((res[i].epoch)*1000).toLocaleTimeString('en-GB');
        var epochVal = epochValDt+' '+epochValTm;

        xAxisTrueSumOfStateVals.push(epochVal);
        xAxisDIASSumOfStateVals.push(epochVal);
        xAxisSumofSelectedStateVals.push(epochVal);

        yAxisTrueSumOfStateVals.push(parseInt(res[i].true_sum_gdelt_events));
        yAxisDIASSumOfStateVals.push(parseInt(res[i].dias_sum_selected_states));
        yAxisSumofSelectedStateVals.push(parseInt(res[i].sum_selected_states));

        xAxisValue<no.>.push(epochVal); //eg. xAxisValue1
        yAxisValue<no.>.push(parseInt(res[i].peer<no.>));
    }

    var trueSumTraces = {
        x: xAxisTrueSumOfStateVals,
        y: yAxisTrueSumOfStateVals,
        name: 'GDELT Actual',
        mode: 'line',
        
        line: {
            dash: 'dot',
            width: 2,
            color: '#3785ba'
        },
        'visible': true,
    };

    var SumStatesTraces = {
        x: xAxisSumofSelectedStateVals,
        y: yAxisSumofSelectedStateVals,
        name: 'DIAS Actual',
        mode: 'line',
        marker: {color: 'rgb(152, 156, 152)'},
        'visible': true,
    }

    var DiasSumTraces = {
        x: xAxisDIASSumOfStateVals,
        y: yAxisDIASSumOfStateVals,
        name: 'DIAS Estimated',
        marker: {color: '#2ca02c'},
        mode: 'line',
        'visible': true,
    }

    var trace<no.> = { //eg. trace1
        x: xAxisValue<no.>, //eg. xAxisValue1
        y: yAxisValue<no.>, //eg. yAxisValue1
        name: countries[<no. -1>].actiongeo_countrycode, //eg countries[0]
        mode: 'markers',
        type: 'scatter',
        'visible': 'legendonly',
    };
