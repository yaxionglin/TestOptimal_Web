

# TestOptimal_MBT
API to access [TestOptimal](https://testoptimal.com) Server for execution.

[TestOptimal](https://testoptimal.com) is a [Model-Based Testing (MBT)](https://en.wikipedia.org/wiki/Model-based_testing) tool, which generates test cases from a state diagram or from a set of variables.



## Usage

You may integrate your web project with //TestOptimal// using javascript library: TOServer.js.

For web/javascript, just add one of the following <script> to HEAD tag in your html page:
    <script src="http://[TestOptimal host:port]/agent/TOServer.js"></script>
    <script src="https://testoptimal.com/v6/agent/TOServer.js"></script>

For //Node.js// project, use following npm command to install //TOServer.js// package:
   npm install @testoptimal/mbt
   

### Connecting to TestOptimal Server
   TOSvr = new TOServer("http://localhost:8888");
   TOSvr.setDebugCB((msg)=> console.logMsg ("DEBUG: " + msg), 2);
   TOSvr.login("myUsername", "myPwd")
      .then(() => {
                console.logMsg("connected to TestOptimal server");
            }, (err) => {
                console.logMsg(err);
                alert("Connection error");
      });

### State Model Sample Scripts


#### Creating State Model
	var model = new Model ("VendingMachine");
	var Welcome = model.addStateInitial("Welcome");
	var Q1 = model.addState("25 cents");
	var Q2 = model.addState("50 cents");
	var Q3 = model.addState("75 cents");
	var Q4 = model.addState("100 cents");
	var ThankYou = model.addStateFinal("ThankYou");
	
	Welcome.addTrans("add_quarter", Q1).addTrans("cancel", ThankYou);
	Q1.addTrans("add_quarter", Q2).addTrans("cancel", ThankYou);
	Q2.addTrans("add_quarter", Q3).addTrans("cancel", ThankYou);
	Q3.addTrans("add_quarter", Q4).addTrans("cancel", ThankYou);
	Q4.addTrans("select_drink", ThankYou).addTrans("cancel", ThankYou);
	
	// TOSvr is the connection object created in Making Connection section above.
	TOSvr.uploadModel(model).then (function(){alert("model uploaded");})

#### Generating Test Cases
	TOSvr.genPaths("VendingMachine", "Optimal", 1000).then(printPaths);
	function printPaths (sum) {
	   console.logMsg(sum);
	}

#### Opening Graphs
	// graph types: model, sequence, msc and coverage
	// except model graph, all other graphs are available for model execution.
	var url = TOSvr.getGraphURL("VendingMachine", "model");
	console.logMsg("model graph: " + url);
	window.open(url, "ModelGraph");

#### Online MBT
	 var execReq = {
		modelName: "DEMO_RemoteAgent",
		statDesc: "description",
		options: { "autoClose": false }
	};
	var agentID = "DEMO";
	TOSvr.execModel(execReq).then((data) => {
		console.logMsg(data);
		console.logMsg("Registering agent " + agentID);	
		TOSvr.regAgent(execReq.modelName, agentID).then(getNextCmd, errHandler);
	});
	
	function getNextCmd() {
	   TOSvr.nextCmd(agentID, 2000).then(function(rmtCmd) {
		  if (rmtCmd && rmtCmd.cmd) {
			  var result = {
				result: "I don't know",
				reqTag: "DEMO", 
				assertID: "DEMO-" + rmtCmd.cmd
			 }
			 console.logMsg("received cmd: " + rmtCmd.cmd);
			 console.logMsg("sending result: " + result.result);
			 TOSvr.setResult (agentID, result).then (function(ret) {
				setTimeout(getNextCmd, 1000);
			  }, errHandler);
		  }
		  else modelExecDone();
	   }, errHandler);
	}
	
	function modelExecDone() {
	   console.logMsg("Model execution completed");
	}function errHandler (err) {
	   console.log("errored");
	   console.logMsg(err);
	   TOSvr.stopModelExec(execReq.modelName);
	}

#### Retrieve Execution Results
   TOSvr.getSummary("DEMO_RemoteAgent").then(function(ret) {console.logMsg(ret);})

### Combinatorial Model Sample Scripts

#### Create DataSet
    var ds = new DataSet("DemoDataSet");
    ds.addField ("F1", "text", ["aa","bbb"], "", false);
    ds.addField ("F2", "int", [1,2,3], "", false);
    TOSvr.uploadDataSet (ds).then(console.logMsg, console.logMsg);

#### Generate Test Cases
   TOSvr.genDataTable("DemoDataSet", "pairWise").then(console.logMsg, console.logMsg);


### Demo Web Client
You may try out above sample scripts with the web client bundled in //TestOptimal// installation:

http://localhost:8886/DemoApp/Demo_MBT.html




## Developing



### Tools

Created with [Nodeclipse](https://github.com/Nodeclipse/nodeclipse-1)
 ([Eclipse Marketplace](http://marketplace.eclipse.org/content/nodeclipse), [site](http://www.nodeclipse.org))   

Nodeclipse is free open-source project that grows with your contributions.
