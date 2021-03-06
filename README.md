## Question Graph

A project that uses a json formatted graph and json formatted questions to navigate conditional paths through a quiz or survey.

['question-tree-core' available on NPM](https://www.npmjs.com/package/question-tree-core)

[Deployed version of this repo](http://calculating-goat.surge.sh/)

[Original Question Tree Project on Github Pages w/ a demo app built in Backbone](https://hansbrough.github.io/question-tree/)

### Overview of the Pieces

* Question Graph json - defines a base path (order) of questions
* Question definitions json - defines the questions themselves as well as optional rules about order.
* 'question-tree-core' package - provides information to your front end web app such as what's the next or previous question.
* Quiz / Survey etc App files - Consumer of the above and files associated with the appearance and behavior of your Quiz. (these files are included here to give an example of how to use the DecisionTree but you can substitute your own versions)

### Graph and Question Json Formatting examples

#### Example Graph File
```javascript
{
  "meta":{
    "graph_id":"1"
  },
  "module_plantClassification":{
    "title":"Plant Classification",
    "questions": [
      {"id":"plantClassification_1"}
    ],
    "next":"module_plantId"
  },
  "module_plantId":{
    "title":"Plant Identification",
    "questions": [
      {"id":"plantId_1", "next":"plantId_2"},
      {"id":"plantId_2", "next":"plantId_3"},
      {"id":"plantId_3"}
    ],
    "next":"module_final"
  },
  "module_final":{
    "title":"Wrapping Up",
    "questions": [
      { "id":"survey_results"}
    ]
  }
}
````
There are a few things to note about the above. First - the questions are grouped by 'modules' - for example there are three defined above: 'plantClassification', 'plantId' and 'final'. This lets authors logically group questions and optionally define paths between modules with the 'next' property. By default the decisionTree will move through modules in order so strictly speaking the module level 'next' properties above are not needed. Next - each module contains a list of question id's which correspond to definitions in the Questions json. Modules and questions in Graph file define a sequential _Base Path_ without branching. Lastly the 'meta' section above will be ignored by the decisionTree when determining order of questions - it's just a place to keep info about the graph itself for example versioning.

#### Example Questions File

````javascript
{
  "plantClassification_01":{
    "title":"What genus does this plant most likely belong to?",
    "media": [
      {"type":"image", "src":"/img/a_celsii.jpg"}
    ],
    "labels":[
      {"title":"Aloe", "qid":"100", "next":"plantClassification_11"},
      {"title":"Agave", "qid":"101"},
      {"title":"Sempervivum", "qid":"102", "next":"plantClassification_13"},
      {"title":"Gasteria", "qid":"103", "next":"plantClassification_14"},
      {"title":"Echeveria", "qid":"104", "next":"plantClassification_12"},
      {"title":"Haworthia", "qid":"105", "next":"plantClassification_15"}
    ],
    "actual":"101",
    "category":"quiz",
    "criterion":"agave",
    "type":"radio"
  },
  "favColors_85":{
    "title":"What colors look good in the kitchen",
    "labels":[
      {"title":"Blue", "qid":"5"},
      {"title":"Green", "qid":"6"},
      {"title":"Yellow", "qid":"7"},
      {"title":"Red", "qid":"8"}
    ],
    "category":"survey",
    "criterion":"kitchen",
    "type":"checkbox"
  }
}
````
Note how the 'plantClassification_01' has been defined above. The 'labels' property is an array of options associated with question. Because this question node is categorized as a 'quiz' there is a correct answer specified with the 'actual' property. The 'type' property indicates that the options should be rendered as radio button controls. The 'criterion' property acts as a metadata tag for use by any application logic needing to summarize user responses. The 'media' property is an array of images etc that may be displayed in the UI.

A deceivingly simple looking property 'next' on label members is how branching can be defined between questions. More specifically _conditional paths_ can be created which will either add to or substract from the Base Path graph length. For example question nodes can be added by specifying a 'next' property which points at a question not defined in the _Base Path_. In other words from the example above the option `{"title":"Sempervivum", "qid":"102", "next":"plantClassification_13"}` will add add the 'plantClassification_13' node to the stack of questions visible to the user (assuming it's defined). Upon completion, unless 'plantClassification_13' itself defines a next question, the user will be returned to the _Base Path_ which in the case of the graph file example would be 'plantId_1' (since its the first question in the next module). Similarly _Shortcut Paths_ can be created by setting a question's 'next' value equal to a node on the _Base Path_ more than a single hop away. For example jumping from 'plantId_1' directly to 'plantId_3'. There are more variations on conditional paths which will be covered later.

Finally any question definitions like 'favColors_85' not referenced in the _Base Path_ or directly from other questions will be ignored.

#### Graph and Questions Helper Files

These files are used by the decisionTree logic to help with tasks like determining which is the current question node and what comes next etc. The Graph and Questions files also handle fetching your json files (via the HTML5 fetch api) and storing the results locally. By design only the decisionTree uses these files, not the view logic which does not need to know about the inner workings of path branching. In general the Graph.js file provides info specific to the graph json while Questions.js does the same for the questions json.

#### DecisionTree

This file combines information about the graph and questions json to determine a path through the question set. It should be instantiated and used by the View logic to determine the 'next' and 'previous' question nodes. The decisionTree logic simply guides the way from the first question to the last it does not try to do other tasks like persisting user answers.

#### View Logic
Use your own presentation files that create the quiz / survey UI or use the React files included here as an example / starting point.

### Path Branching Overview

Below is a listing of path type examples from simple to more complex. The term _Graph Length_ refers to the total number of nodes encountered by a user from start to finish during quiz or survey. It can include from one to many paths e.g. a simple, sequential path or a path which includes several conditional branches.

_**Base Path**_ is the sequential route through nodes in a graph which avoids any conditional paths. The diagram below has a _Graph Length_ of 4 nodes. This simplest type of path is defined in the Graph json file.

![Image of Base Path](https://user-images.githubusercontent.com/658255/28881940-98624038-775e-11e7-9a4e-c1672a51cd81.png)


_**Conditional Path**_ - an optional, offshoot path that exposes users to extra questions or allows skipping questions in the Base Path. Conditional paths are defined with the 'next' property in the Questions json file.

_**Detour Path**_ - a type of _Conditional Path_ which exposes users to _additional_ questions. In the diagram below if a user took the optional detour branch then the _Graph Length_ is 4 nodes else the _Graph Length_ would be 3 nodes.

![Image of Detour Path](https://user-images.githubusercontent.com/658255/28882908-7f1ab4d6-7761-11e7-8c29-af40f5cd2af2.png)

_**Shortcut Path**_ - a type of _Conditional Path_ that allows users to bypass questions on the _Base Path_. In the diagram below the _Graph Length_ is only 2 nodes if the shortcut is used.

![Image of Shortcut Path](https://user-images.githubusercontent.com/658255/28883099-294ebac4-7762-11e7-8e54-0b982504954f.png)

_**Multi-Node Path**_ - path containing more than one node. Almost all Sequential paths will be multi-node. Conditional Paths might often contain just a single node. The diagram below shows a multi-node detour path with a graph length of 5 nodes.

![Image of MultiNode Path](https://user-images.githubusercontent.com/658255/28883656-0bdc5f26-7764-11e7-8308-d2ccc0bae480.png)

_**Multi-Branch Path**_ - node from which multiple, conditional paths are available to choose from.

![Image of MultiBranch Path](https://user-images.githubusercontent.com/658255/28884089-69aa222c-7765-11e7-8b3e-12ab5f657393.png)

_**Compound Conditional Path**_ - A Conditional Path with nested, conditional paths.

![Image of Compound Path](https://user-images.githubusercontent.com/658255/28884380-6d912060-7766-11e7-8ab9-da45c038dab2.png)

_**Mixed Conditional Path**_ - A compound conditional path that contains both detour and shortcut paths. In the diagram below 'path 1' represents a _Shortcut Path_ with a _Graph Length_ of 3, while 'path 2' represents a _Detour Path_ with a _Graph Length_ of 5.

![Image of Mixed Path](https://user-images.githubusercontent.com/658255/28886182-624b6d4e-776d-11e7-995a-c9047e55d185.png)
