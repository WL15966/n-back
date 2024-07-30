
  
### n-back 实验设计教程  

下面是一个详细的jsPsych n-back实验设计教程，适合对jspsych完全不了解的新手。我将逐步解释代码文件中的内容，并指导你如何复制和完成n-back实验设计。  
  

### 1. 简介  
  

**jsPsych** 是一个用于创建和运行心理学、神经科学和心理语言学实验的JavaScript库。JsPsych允许通过网页浏览器（比如Edge、Google、火狐等）进行在线实验。  
  

**n-back实验** 是测试工作记忆的实验，要求参与者记住一系列信息，并在特定条件下做出反应。  
  

### 2. 准备环境  
  

首先，需要一个可以运行HTML和JavaScript的环境。可以使用网页浏览器（如Edge、Google、火狐等）来运行jsPsych实验。  
  

### 3. 引入jsPsych库  
  

在HTML文件中，需要引入jsPsych库及其插件。以下是代码文件中的相关部分：  
  

```html  
<!DOCTYPE html>  
<html>  
  <head>  
    <script src="jspsych/jspsych.js"></script>  
    <script src="jspsych/plugins/jspsych-html-keyboard-response.js"></script>  
    <script src="jspsych/plugins/jspsych-html-button-response.js"></script>  
    <link rel="stylesheet" href="jspsych/jspsych.css">  
  </head>  
  <body>  
  </body>  
</html>  
```  
  

- `jspsych.js` 是jsPsych的核心库。  
- `jspsych-html-keyboard-response.js` 和 `jspsych-html-button-response.js` 是jsPsych的插件，分别用于键盘响应和按钮响应。  
- `jspsych.css` 是jsPsych的样式文件。  
  

### 4. 初始化实验  
  

在`<script>`标签中，初始化jsPsych实验：  
  

```javascript  
<script>  
  var timeline = [];  
  
  var n_back_set = ['Z', 'X', 'C', 'V', 'B', 'N'];  
  var sequence = [];  
  
  var how_many_back = 2;  
  var sequence_length = 22;  
  
  jsPsych.init({  
    timeline: timeline  
  });  
</script>  
```  
  

- `timeline` 是一个数组，用于存储实验的各个部分。  
- `n_back_set` 是一个包含可能显示的字母的数组。  
- `sequence` 是一个空数组，用于存储实验过程中的字母序列。  
- `how_many_back` 定义了n-back任务中的“n”，这里是2-back。  
- `sequence_length` 定义了序列的长度。  
  

### 5. 实验指导  
  

在实验开始前，向参与者提供实验指导。以下是代码文件中的相关部分：  
  

```javascript  
var instructions_1 = {  
  type: 'html-button-response',  
  stimulus: '<div style="width: 800px;">'+  
    '<p>This experiment will test your ability to hold information in short-term, temporary memory. This is called working memory.</p>'+  
    '</div>',  
  choices: ["Continue"]  
}  
timeline.push(instructions_1);  
  
var instructions_2 = {  
  type: 'html-button-response',  
  stimulus: '<div style="width: 800px;">'+  
    '<p>You will see a sequence of letters presented one at a time. Your task is to determine if the letter on the screen matches '+  
    'the letter that appeared two letters before.</p>'+  
    '<p>If the letter is match <span style="font-weight: bold;">press the M key.</span></p>'+  
    '<p>For example, if you saw the sequence X, C, V, B, V, X you would press the M key when the second V appeared on the screen.</p>'+  
    '<p>You do not need to press any key when there is not a match.</p>'+  
    '</div>',  
  choices: ["Continue"]  
}  
timeline.push(instructions_2);  
  
var instructions_3 = {  
  type: 'html-button-response',  
  stimulus: '<div style="width: 800px;">'+  
    '<p>The sequence will begin on the next screen.</p>'+  
    '<p>Remember: press the M key if the letter on the screen matches the letter that appeared two letters ago.</p>'+  
    '</div>',  
  choices: ["I'm ready to start!"],  
  post_trial_gap: 1000  
}  
timeline.push(instructions_3);  
```  
  

### 6. n-back任务  
  

定义n-back任务的试验：  
  

```javascript  
var n_back_trial = {  
  type: 'html-keyboard-response',  
  stimulus: function() {  
    if(sequence.length < how_many_back){  
      var letter = jsPsych.randomization.sampleWithoutReplacement(n_back_set, 1)[0]  
    } else {  
      if(jsPsych.timelineVariable('match', true) == true){  
        var letter = sequence[sequence.length - how_many_back];  
      } else {  
        var possible_letters = jsPsych.randomization.sampleWithoutReplacement(n_back_set, 2);  
        if(possible_letters[0] != sequence[sequence.length - how_many_back]){  
          var letter = possible_letters[0];  
        } else {  
          var letter = possible_letters[1];  
        }  
      }  
    }  
    sequence.push(letter);  
    return '<span style="font-size: 96px;">'+letter+'</span>'  
  },  
  choices: ['M'],  
  trial_duration: 1500,  
  response_ends_trial: false,  
  post_trial_gap: 500,  
  data: {  
    phase: 'test',  
    match: jsPsych.timelineVariable('match')  
  },  
  on_finish: function(data){  
    if(data.match == true){  
      data.correct = (data.key_press != null)  
    }  
    if(data.match == false){  
      data.correct = (data.key_press === null)  
    }  
  }  
}  
  
var n_back_trials = [  
  {match: true},  
  {match: false}  
]  
  
var n_back_sequence = {  
  timeline: [n_back_trial],  
  timeline_variables: n_back_trials,  
  sample: {  
    type: 'with-replacement',  
    size: sequence_length,  
    weights: [1, 2]  
  }  
}  
  
timeline.push(n_back_sequence);  
```  
  

- `n_back_trial` 定义了一个试验，其中包含字母的显示和响应的逻辑。  
- `match` 变量用于确定当前试验是否为匹配试验。  
- `sequence` 数组用于存储字母序列，并在需要时检查匹配。  
- `n_back_sequence` 定义了整个n-back任务的序列。  
  

### 7. 反馈  
  

实验结束后，向参与者提供结果反馈：  
  

```javascript  
var feedback = {  
  type: 'html-keyboard-response',  
  stimulus: function(){  
    var test_trials = jsPsych.data.get().filter({phase: 'test'}).last(sequence_length-2);  
    var n_match = test_trials.filter({match: true}).count();  
    var n_nonmatch = test_trials.filter({match: false}).count();  
    var n_correct = test_trials.filter({match: true, correct: true}).count();  
    var false_alarms = test_trials.filter({match: false, correct: false}).count();  
  
    var html = "<div style='width:800px;'>"+  
      "<p>All done!</p>"+  
      "<p>You correctly identified "+n_correct+" of the "+n_match+" matching items.</p>"+  
      "<p>You incorrectly identified "+false_alarms+" of the "+n_nonmatch+" non-matching items as matches.</p>"  
    return html;  
  },  
  choices: jsPsych.NO_KEYS  
}  
timeline.push(feedback);  
```  
  

- `feedback` 显示了参与者在整个实验中的表现。  
  

### 8. 运行实验  
  

将以上代码保存为HTML文件，并在浏览器中打开。你将看到一个完整的n-back实验，参与者可以通过键盘响应完成任务。  
  

### 9. 总结  
  

通过以上步骤，你可以创建一个基本的jsPsych n-back实验。你也可以根据自己的需要调整实验参数，如字母集、序列长度等。希望这个教程对你有所帮助！  
  

