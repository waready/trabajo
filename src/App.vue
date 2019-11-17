<template>
  <div id="app">
   <nav class="navbar navbar-dark bg-dark">
     <a class="navbar-brand" href="#">Arbol de relacion</a>
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarNavDropdown">
        <ul class="navbar-nav">
          <li class="nav-item active">
            <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#">Features</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#">Pricing</a>
          </li>
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownMenuLink" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
              Dropdown link
            </a>

          </li>
        </ul>
      </div>
    </nav> 
    <div class="container">
      <div class="row">
        <div class="col-md-3">
          <div class="jumbotron">
            <h1 class="display-4">Lista</h1>
            <ul class="list-group">
              <li class="list-group-item "><a id="fire_tennis" href="#">cielo</a></li>
              <li class="list-group-item"><a id="fire_voting" href="#">autos</a></li>
            </ul>
          </div>

        </div>

        <div class="col-md-9">
          <h3 class="text-center">Arbol</h3>  
          <div id="data-container">
          <div  id='canvas'></div>
          <!-- Button trigger modal -->
          <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModalLong">
            ver tabla de datos
          </button>

          <!-- Modal -->
          <div class="modal fade" id="exampleModalLong" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title" id="exampleModalLongTitle"> Data</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                  </button>
                </div>
                <div class="modal-body">
                  <table id='training'>
                  </table>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>

                </div>
              </div>
            </div>
          </div>
                      <!-- <h3>Sample Predictions</h3>
            <table id='samples'>
            </table> -->

          
          </div>
        </div>
        </div>



      </div>
       
  </div>
</template>

<script>

var id3 = function(_s,target,features){
    var targets = _.unique(_s.pluck(target));
    if (targets.length == 1){
	console.log("end node! "+targets[0]);
	return {type:"result", val: targets[0], name: targets[0],alias:targets[0]+randomTag() }; 
    }
    if(features.length == 0){
	console.log("returning the most dominate feature!!!");
	var topTarget = mostCommon(_s.pluck(target));
	return {type:"result", val: topTarget, name: topTarget, alias: topTarget+randomTag()};
    }
    var bestFeature = maxGain(_s,target,features);
    var remainingFeatures = _.without(features,bestFeature);
    var possibleValues = _.unique(_s.pluck(bestFeature));
    console.log("node for "+bestFeature);
    var node = {name: bestFeature,alias: bestFeature+randomTag()};
    node.type = "feature";
    node.vals = _.map(possibleValues,function(v){
	console.log("creating a branch for "+v);
	var _newS = _(_s.filter(function(x) {return x[bestFeature] == v}));
	var child_node = {name:v,alias:v+randomTag(),type: "feature_value"};
	child_node.child =  id3(_newS,target,remainingFeatures);
	return child_node;
	
    });
    return node;
}

var predict = function(id3Model,sample) {
    var root = id3Model;
    while(root.type != "result"){
	var attr = root.name;
	var sampleVal = sample[attr];
	var childNode = _.detect(root.vals,function(x){return x.name == sampleVal});
	root = childNode.child;
    }
    return root.val;
}

//necessary math functions

var entropy = function(vals){
    var uniqueVals = _.unique(vals);
    var probs = uniqueVals.map(function(x){return prob(x,vals)});
    var logVals = probs.map(function(p){return -p*log2(p) });
    return logVals.reduce(function(a,b){return a+b},0);
}

var gain = function(_s,target,feature){
    var attrVals = _.unique(_s.pluck(feature));
    var setEntropy = entropy(_s.pluck(target));
    var setSize = _s.size();
    var entropies = attrVals.map(function(n){
	var subset = _s.filter(function(x){return x[feature] === n});
	return (subset.length/setSize)*entropy(_.pluck(subset,target));
    });
    var sumOfEntropies =  entropies.reduce(function(a,b){return a+b},0);
    return setEntropy - sumOfEntropies;
}

var maxGain = function(_s,target,features){
    return _.max(features,function(e){return gain(_s,target,e)});
}

var prob = function(val,vals){
    var instances = _.filter(vals,function(x) {return x === val}).length;
    var total = vals.length;
    return instances/total;
}

var log2 = function(n){
    return Math.log(n)/Math.log(2);
}

var mostCommon = function(l){
   return  _.sortBy(l,function(a){
	return count(a,l);
    }).reverse()[0];
}

var count = function(a,l){
    return _.filter(l,function(b) { return b === a}).length
}

var randomTag = function(){
    return "_r"+Math.round(Math.random()*1000000).toString();
}

//Display logic

var drawGraph = function(id3Model,divId){
    var g = new Array();
    g = addEdges(id3Model,g).reverse();
    window.g = g;
    var data = google.visualization.arrayToDataTable(g.concat(g));
    var chart = new google.visualization.OrgChart(document.getElementById(divId));
    google.visualization.events.addListener(chart, 'ready',function(){
    _.each($('.google-visualization-orgchart-node'),function(x){
	var oldVal = $(x).html();
	if(oldVal){
	    var cleanVal = oldVal.replace(/_r[0-9]+/,'');
	    $(x).html(cleanVal);
	}
}); 
    });
    chart.draw(data, {allowHtml: true});

}

var addEdges = function(node,g){
    if(node.type == 'feature'){
	_.each(node.vals,function(m){
	    g.push([m.alias,node.alias,'']);
	    g = addEdges(m,g);
	});
	return g;
    }
    if(node.type == 'feature_value'){

	g.push([node.child.alias,node.alias,'']);
	if(node.child.type != 'result'){
	    g = addEdges(node.child,g);
	}
	return g;
    }
    return g;
}


var renderSamples = function(samples,$el,model,target,features){
    _.each(samples,function(s){
	var features_for_sample = _.map(features,function(x){return s[x]});
	$el.append("<tr><td>"+features_for_sample.join('</td><td>')+"</td><td><b>"+predict(model,s)+"</b></td><td>actual: "+s[target]+"</td></tr>");
    })
}

var renderTrainingData = function(_training,$el,target,features){
    _training.each(function(s){
	$el.append("<tr><td>"+_.map(features,function(x){return s[x]}).join('</td><td>')+"</td><td>"+s[target]+"</td></tr>");
    })
}

var calcError = function(samples,model,target){
    var total = 0;
    var correct = 0;
    _.each(samples,function(s){
	total++;
	var pred = predict(model,s);
	var actual = s[target];
	if(pred == actual){
	    correct++;
	}
    });
    return correct/total;
}

$(document).ready(function(){ 
  
      console.log('all systems go');
      $("#fire_tic_tac_toe").click(function(e){
           e.preventDefault();
           var testModel = id3(examples_tic_tac_toe,'classification',features_tic_tac_toe);
           drawGraph(testModel,'canvas');
           renderSamples(samples_tic_tac_toe,$("#samples"),testModel,'classification',features_tic_tac_toe);
           renderTrainingData(examples_tic_tac_toe,$("#training"),'classification',features_tic_tac_toe);
});

     $("#fire_voting").click(function(e){
           e.preventDefault();
           var testModel = id3(autos,'rentable',features_autos);
           drawGraph(testModel,'canvas');
           renderSamples(voting_samples,$("#samples"),testModel,'rentable',features_autos);
           renderTrainingData(autos,$("#training"),'rentable',features_autos);
});


$("#fire_tennis").click(function(e){
           e.preventDefault();
           var testModel = id3(examples,'play',features);
           drawGraph(testModel,'canvas');
           renderSamples(samples,$("#samples"),testModel,'play',features);
           renderTrainingData(examples,$("#training"),'play',features);
      console.log("error");
      console.log(calcError(samples,testModel,'play'));

      });
});


var examples = [
{day:'D1',outlook:'Sunny', temp:'Hot', humidity:'High', wind: 'Weak',play:'No'},
{day:'D2',outlook:'Sunny', temp:'Hot', humidity:'High', wind: 'Strong',play:'No'},
{day:'D3',outlook:'Overcast', temp:'Hot', humidity:'High', wind: 'Weak',play:'Yes'},
{day:'D4',outlook:'Rain', temp:'Mild', humidity:'High', wind: 'Weak',play:'Yes'},
{day:'D5',outlook:'Rain', temp:'Cool', humidity:'Normal', wind: 'Weak',play:'Yes'},
{day:'D6',outlook:'Rain', temp:'Cool', humidity:'Normal', wind: 'Strong',play:'No'},
{day:'D7',outlook:'Overcast', temp:'Cool', humidity:'Normal', wind: 'Strong',play:'Yes'},
{day:'D8',outlook:'Sunny', temp:'Mild', humidity:'High', wind: 'Weak',play:'No'},
{day:'D9',outlook:'Sunny', temp:'Cool', humidity:'Normal', wind: 'Weak',play:'Yes'},
{day:'D10',outlook:'Rain', temp:'Mild', humidity:'Normal', wind: 'Weak',play:'Yes'},
{day:'D11',outlook:'Sunny', temp:'Mild', humidity:'Normal', wind: 'Strong',play:'Yes'},
{day:'D12',outlook:'Overcast', temp:'Mild', humidity:'High', wind: 'Strong',play:'Yes'},
{day:'D13',outlook:'Overcast', temp:'Hot', humidity:'Normal', wind: 'Weak',play:'Yes'},
{day:'D14',outlook:'Rain', temp:'Mild', humidity:'High', wind: 'Strong',play:'No'}
];

examples = _(examples);

var features = ['outlook', 'temp', 'humidity', 'wind'];
//no importa
var samples = [
	{outlook:'Overcast', temp:'Mild', humidity:'High', wind: 'Strong',play: 'Yes'},
	{outlook:'Rain', temp:'Mild', humidity:'High', wind: 'Strong', play: 'No'},
	{outlook:'Sunny', temp:'Cool', humidity:'Normal', wind: 'Weak', play: 'Yes'}
]


var autos = [
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	0},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	'<=64312'	,year:	'<=2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	0},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	0	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	0},
	{km:	'<=64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	0	,rentable:	1},
	{km:	' >64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	0},
	{km:	' >64312'	,year:	' >2012'	,price:	' >12744'	,marca:	1	,rentable:	1},
	{km:	'<=64312'	,year:	' >2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1},
	{km:	' >64312'	,year:	'<=2012'	,price:	'<=12744'	,marca:	1	,rentable:	1}
	


];		

autos = _(autos);		

var features_autos = ['km', 'year', 'price','marca'];
var samples = [
	{outlook:'Overcast', temp:'Mild', humidity:'High', wind: 'Strong',play: 'Yes'},
	{outlook:'Rain', temp:'Mild', humidity:'High', wind: 'Strong', play: 'No'},
	{outlook:'Sunny', temp:'Cool', humidity:'Normal', wind: 'Weak', play: 'Yes'}
]


export default {
  name: 'app'
}
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

main {
  text-align: center;
  margin-top: 40px;
}

header {
  margin: 0;
  height: 56px;
  padding: 0 16px 0 24px;
  background-color: #35495E;
  color: #ffffff;
}

header span {
  display: block;
  position: relative;
  font-size: 20px;
  line-height: 1;
  letter-spacing: .02em;
  font-weight: 400;
  box-sizing: border-box;
  padding-top: 16px;
}
</style>
