<template>
    <div class="content" id="EvolutionViewSVG">
         <span>{{ message }}</span>
    </div>
           
</template>
<script>
// import NetService from '../../services/net-service';
// import DataService from '../../services/data-service';
import * as d3 from 'd3';

export default {
    name: 'EvolutionView',
    data() {
        return {
            width: null,
            height: null,
            svg: null,
            newMessage:null,
        };
    },
    props: ['message'],
    mounted() {
        this.newMessage = this.message;
        this.drawThemeRiver();
        // this.drawEventDrop(self.svg);      
    },
    methods: { 
        drawThemeRiver(){
            
            const path = 'http://localhost:5003/data';

            chart(path);

            var datearray = [];
            var Statuscolor = [];
            var Langcolor = [];

            function chart(csvpath) {


                Statuscolor = ["#99c6ff", "#4291f7", "#2f7ce0","#206ac9", "#1357ad", "#084591","#1b1ba1","#000061"];

                Langcolor = ["#e6b8af", "#ffd966", "#b6d7a8","#9fc5e8", "#b4a7d6", "#c27ba0","#d6c6a7"];


                var strokecolor = 'black';

                var formatmonth = d3.timeParse("%Y-%m");

                var margin = {
                    top: 20,
                    right: 40,
                    bottom: 30,
                    left: 50
                };
                var width = 700 - margin.left - margin.right;
                var height = 400 - margin.top - margin.bottom;

                var tooltip = d3.select('#EvolutionViewSVG')
                    .append("div")
                    .attr("class", "remove")
                    .style("position", "absolute")
                    .style("z-index", "20")
                    .style("visibility", "hidden")
                    .style("top", "80px")
                    .style("width", "200px")
                    .style("left", "130px");

                var x = d3.scaleTime()
                    .range([0, width]);
                var y = d3.scaleLinear()
                var yverse = d3.scaleLinear(); 
                var ztop = d3.scaleOrdinal()
                    .range(Statuscolor);
                var zdown = d3.scaleOrdinal()
                    .range(Langcolor);

                var nest = d3.nest()
                    .key(function(d) {
                        return d.time;
                    });

                var areatop = d3.line()
                    .curve(d3.curveMonotoneX)
                    .x(function(d) {
                        return x(new Date(d.data.key));
                    })
                    .y(function(d) {
                        if(isNaN(d[1])){return y(d[0])}
                        else{return y(d[1]);}
                    });
                var areadown = d3.line()
                    .curve(d3.curveMonotoneX)
                    .x(function(d) {
                        return x(new Date(d.data.key));
                    })
                    .y(function(d) {
                        return y(d[0]);                       
                    });
                
                var areatop2 = d3.line()
                    .curve(d3.curveMonotoneX)
                    .x(function(d) {
                        return x(new Date(d.data.key));
                    })
                    .y(function(d) {
                        if(isNaN(d[1])){return yverse(d[0])}
                        else{return yverse(d[1]);}
                    });
                var areadown2 = d3.line()
                    .curve(d3.curveMonotoneX)
                    .x(function(d) {
                        return x(new Date(d.data.key));
                    })
                    .y(function(d) {
                        return yverse(d[0]);                       
                    });

                var svg = d3.select("#EvolutionViewSVG")
                            .append("svg")
                            .attr('class', 'svg')
                            .attr("width", width + margin.left + margin.right)
                            .attr("height", height + margin.top + margin.bottom)
                            .append("g")
                            .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

                
                function getrequest(url, callback){
                    var xhr = new XMLHttpRequest()
                    xhr.onreadystatechange = function(){
                        if(xhr.readyState == XMLHttpRequest.DONE){
                            callback(eval(xhr.responseText))
                        }
                    }
                    xhr.open("GET", csvpath, true)
                    xhr.send(null)
                }
                
                getrequest(csvpath, draw); 
                        
                function draw(data){
                    //console.log(data[1])
                    data[0].forEach(function(d) {
                        // d.author = d.author
                        d.time = formatmonth(d.time);
                        d.judgeStatus = d.judgeStatus;
                        d.judgeStatusnum = +d.judgeStatusnum;

                    });
                    data[1].forEach(function(d) {
                        // d.author = d.author
                        d.time = formatmonth(d.time);
                        d.language = d.language
                        d.langnum= +d.langnum;
                    });

                    // var useronename = data[0].author;                    
                    // var userone = [];
                    // var usertwo = [];
                    // var useroneindex = 0;
                    // var usertwoindex = 0;
                    // for(var reclen=0; reclen<data.length;reclen++){
                        
                    //     if(data[reclen].author==useronename){
                    //         userone[useroneindex] = data[reclen];
                    //         useroneindex++;
                    //     }else{
                    //         usertwo[usertwoindex] = data[reclen];
                    //         usertwoindex++;
                    //     }
                    // }
                    
                    var stackStatus = d3.stack()
                        .keys(function() {
                            var findkey = nest.entries(data[0]);
                            //console.log(findkey)
                            var key = [];
                            for (var g = 0; g < findkey[0].values.length; g++) {
                                key[g] = findkey[0].values[g].judgeStatus;
                            }
                            //console.log(key);
                            return key;
                        })
                        .order(d3.stackOrderNone)
                        .offset(d3.stackOffNone);
                    var stackLang = d3.stack()
                        .keys(function() {
                            // var findkey = nest.entries(data[0]);
                            // console.log(findkey)
                            // var key = [];
                            // for (var g = 0; g < findkey[0].values.length; g++) {
                            //     key[g] = findkey[0].values[g].key;
                            // }
                            // console.log(key);
                            return ['C','C#','C++',"G++",'GCC','Java',"Pascal"];
                        })
                        .order(d3.stackOrderNone)
                        .offset(d3.stackOffNone);

                    var nestresultStatus = nest.entries(data[0]);
                    var nestresultLang = nest.entries(data[1]);

                    //console.log(nestresultLang);
                    
                    for (var i = 0; i < nestresultStatus.length; i++) {
                        var obj = nestresultStatus[i].values;
                        delete nestresultStatus[i].values;
                        delete nestresultStatus[i].value;
                        for (var j = 0; j < obj.length; j++) {
                            nestresultStatus[i][obj[j].judgeStatus] = obj[j].judgeStatusnum;
                        }
                    };
                    for (var i = 0; i < nestresultLang.length; i++) {
                        var obj = nestresultLang[i].values;
                        delete nestresultLang[i].values;
                        delete nestresultLang[i].value;
                        for (var j = 0; j < obj.length; j++) {
                            nestresultLang[i][obj[j].language] = obj[j].langnum;
                        }
                    };

                    var layersone = stackStatus(nestresultStatus);
                    var layerstwo = stackLang(nestresultLang);

                    x.domain(d3.extent(data[0], function(d) {
                        return d.time;
                    }));

                    y.domain([
                        d3.min(layersone, function(layer) {
                            return d3.min(layer, function(d) {
                                return d[0];
                            })
                        }), 
                        d3.max(layersone, function(layer) {
                            return d3.max(layer, function(d) {
                                return d[1];
                            })
                        })
                        ])
                        .range([height/2, 0]);

                    yverse.domain([
                        d3.min(layerstwo, function(layer) {
                            return d3.min(layer, function(d) {
                                return d[0];
                            })
                        }), 
                        d3.max(layerstwo, function(layer) {
                            return d3.max(layer, function(d) {
                                return d[1];
                            })
                        })])
                        .range([height/2, height]);

                    for(var diffindex = 0; diffindex<layersone.length; diffindex++){
                        svg.selectAll(".layer")
                            .data([layersone[layersone.length-1-diffindex]])
                            .enter().append("path")
                            .attr("class", "min_line")
                            .attr("d", function(d) {
                                var changeline = areatop(d);
                                var versearr = [];
                                for(var g=0; g<d.length;g++){
                                    
                                    versearr[g] = d[d.length-1-g];
                                }
                                var changelinedown = areadown(versearr)
                                var firstel = changelinedown.split('M')
                                firstel.splice(0,1)
                                var down = 'L'+ firstel
                                var back =changeline + down
                                return back;
                            })
                            .attr("fill", function(d,i) {
                                return ztop(d.index);
                            });
                    }

                    for(var diffindextwo = 0; diffindextwo<layerstwo.length; diffindextwo++){
                        svg.selectAll(".layer")
                            .data([layerstwo[layerstwo.length-1-diffindextwo]])
                            .enter().append("path")
                            .attr("class", "min_line")
                            .attr("d", function(d) {
                                var changeline = areatop2(d);
                                var versearr = [];
                                for(var g=0; g<d.length;g++){
                                    
                                    versearr[g] = d[d.length-1-g];
                                }
                                var changelinedown = areadown2(versearr)
                                var firstel = changelinedown.split('M')
                                firstel.splice(0,1)
                                var down = 'L'+ firstel
                                var back =changeline + down
                                return back;
                            })
                            .attr("fill", function(d,i) {
                                return zdown(d.index);
                            });
                }
                    
                    svg.append("g")
                        .attr("class", "x axis")
                        .attr("transform", "translate(0," + height + ")")
                        .call(d3.axisBottom(x));

                    svg.append("g")
                        .attr("class", "y axis")
                        .attr("transform", "translate(" + width + ", 0)")
                        .call(d3.axisRight(y));

                    svg.append("g")
                        .attr("class", "y axis")
                        .call(d3.axisLeft(y));

                    svg.append("g")
                        .attr("class", "y axis")
                        .attr("transform", "translate(" + width + ", 0)")
                        .call(d3.axisRight(yverse));

                    svg.append("g")
                        .attr("class", "y axis")
                        .call(d3.axisLeft(yverse));

                    svg.selectAll(".min_line")
                        .attr("opacity", 1)
                        .on("mouseover", function(d, i) {
                            svg.selectAll(".min_line").transition()
                                .duration(250)
                                .attr("opacity", function(d, j) {
                                    return j != i ? 0.1 : 1;
                                })
                            var mousex = d3.mouse(this);
                            
                            mousex = mousex[0];
                            var invertedx = x.invert(mousex);
                            
                            invertedx = invertedx.getFullYear() + invertedx.getMonth();
                            var selected = (d);

                            for (var k = 0; k < selected.length; k++) {
                                datearray[k] = new Date(selected[k].data.key)
                                var mid = datearray[k]
                                
                                datearray[k] = mid.getFullYear()+mid.getMonth();
                            }

                            try {
                                //tryCode - 尝试执行代码块
                                var mousedate = datearray.indexOf(invertedx);
                                var pro = d[mousedate].data[d.key];//error
                                //console.log(d[mousedate])
                                d3.select('.remove')
                                .text(d.key + ': ' + pro)
                                .style("visibility", "visible");    
                            }
                            catch(err) {
                                //catchCode
                                document.getElementsByClassName("min_line").innerHTML = err.message;
                            } 
                        })
                        .on("mouseout", function(d, i) {
                            var pro = [];
                            svg.selectAll(".min_line")
                                .transition()
                                .duration(250)
                                .attr("opacity", "1");
                            d3.select('.remove')
                                .style("visibility", "hidden")
                        })

                    // var vertical = svg.append("path")
                    //     .attr("d", "M0,0L0,350")
                    //     .attr("stroke","black")
                    //     .attr("stroke-width", "1px")

                    // svg.on("mousemove", function(d) {
                    //         var mousex = d3.mouse(this);
                    //         vertical.attr("transform",function(){
                    //             return "translate("+mousex[0]+","+0+")"
                    //         })
                    //     })
                    //     .on("mouseover", function(d) {
                    //         var mousex = d3.mouse(this);
                    //         vertical.attr("transform",function(){                            
                    //             return "translate("+mousex[0]+","+mousex[1]+")"
                    //         })
                    //     });
                }
        }
    },
    
    drawEventDrop(svg){

        const path = 'http://localhost:5003/jsondata';
        d3.json(path, function(error, newdata){
             if(error){
                 console.log(error);
                 return;
             }
            var each_author
            // console.log(newdata);

            var colormap = {
                "Wrong Answer":'#e3893a',
                "Runtime Error":'#e3893a',
                "Presentation Error":'#e3893a',
                "Compilation Error":'#e3893a',
                "Memory Limit Exceeded":'#e3893a',
                "Time Limit Exceeded":'#e3893a',
                "Output Limit Exceeded":'#e3893a',
                "Accepted": "#2952ae"
            };

            var repositoriesData = []
            // for(each_author in newdata){
            //     var new_record = {}
            //     new_record['name'] = each_author
            //     new_record['fullData'] = []
            //     new_record['data'] = []
            // //    console.log(newdata[each_author])
            //     for(let j = 0; j < newdata[each_author].length; j++){
            //     	let temp = {}
            //         temp['date'] = new Date(newdata[each_author][j]['time'])
            //         // console.log(temp['date'])
			// 		temp['color']= colormap[newdata[each_author][j]["judgeStatus"]];
            //         new_record['fullData'].push(temp)
            //         new_record['data'].push(temp)
			// 	}
            //     repositoriesData.push(new_record)
            // }

            var chart = eventDrops({
                range: {
                    start: new Date('01/01/2017 00:00:00 AM'),
                    end: new Date('01/01/2018 00:00:00 AM')
                },
                drop: {
					date: d => d.date,
					color: d => d.color,
                },
            });
            
            repositoriesData = [
                {
                   "0": [{
                    "date" : "02/01/2017 00:00:00 AM",
                    "color" : "red"
                    },
                    {
                        "date" : "03/01/2017 00:00:00 AM",
                    "color" : "red"
                    },
                    {
                    "date" : "04/01/2017 00:00:00 AM",
                    "color" : "red"
                    }]
                }
            ]
            // console.log(repositoriesData)
			d3.select("#EvolutionViewSVG").data([repositoriesData]).call(chart);
             });

    }

 }
}
</script>

<style scoped>
    body {
        font: 10px sans-serif;
    }
    
    .chart {
        background: #fff;
    }
    
    p {
        font: 12px helvetica;
    }
    
    .axis path,
    .axis line {
        fill: none;
        stroke: #000;
        stroke-width: 2px;
        shape-rendering: crispEdges;
    }
    
    button {
        position: absolute;
        right: 50px;
        top: 10px;
    }
    .container {
        text-align: center;
    }
    .card-box {
        padding: 20px;
        background-color: #ffffff;
        box-shadow: 0 8px 42px 0 rgba(0, 0, 0, 0.08);
        border-radius: 5px;
        background-clip: padding-box;
        margin-bottom: 20px;
        width: 40%;
        text-align: center;
        display: inline-block;
    }
	.topbar .top-bar-left {
        float: left;
        position: relative;
        width: 180px;
        z-index: 1;
        background-color: #ffffff;
    }
    .logo {
        color: #ff606f !important;
        font-size: 18px;
        font-weight: 700;
        letter-spacing: .05em;
        line-height: 60px;
    }
    button:hover {
        opacity: 0.8;
    }
    button {
        background-color: #fe6271;
        color: white;
        padding: 14px 20px;
        margin: 8px 0;
        border: none;
        cursor: pointer;
        width: 100%;
    }
    input[type=text], input[type=password] {
        width: 100%;
        padding: 12px 20px;
        margin: 8px 0;
        display: inline-block;
        border: 1px solid rgb(255, 255, 255);
        box-sizing: border-box;
    }
</style>
